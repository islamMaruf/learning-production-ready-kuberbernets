# Tutorial 4: Terraform Infrastructure Deployment

## 🎯 Learning Objectives
- Understand Terraform project structure
- Create VPC with Terraform modules
- Deploy EKS cluster with Auto Mode
- Configure networking and security groups
- Manage Terraform state
- Use Terraform outputs for next steps

## 📋 Prerequisites
- Completed Tutorial 3 (all tools installed)
- AWS CLI configured with valid credentials
- Terraform initialized in project directory

---

## Part 1: Understanding the Terraform Project Structure

### Project Layout

```
terraform/
├── main.tf              # Main configuration file
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── providers.tf         # Provider configurations
├── terraform.tfvars     # Variable values (optional)
├── modules/
│   ├── vpc/            # VPC module
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── eks/            # EKS module
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── .terraform/         # Downloaded providers (auto-generated)
```

### What Each File Does

**main.tf**
- Central configuration
- Calls modules
- Defines resources
- Connects components

**variables.tf**
- Declares input parameters
- Defines defaults
- Sets validation rules
- Documents expected inputs

**outputs.tf**
- Exports values
- Used by other tools
- Displayed after apply
- Can reference resources

**providers.tf**
- Configures AWS provider
- Sets region
- Defines authentication
- Specifies versions

---

## Part 2: Setting Up AWS Backend (Optional but Recommended)

### Why Use Remote Backend?

**Without Remote Backend (Local):**
- State stored on your computer
- Lost if computer crashes
- Can't collaborate easily
- No locking mechanism

**With Remote Backend (S3):**
- State stored in AWS S3
- Versioned and backed up
- Team can collaborate
- DynamoDB locking prevents conflicts

### Create S3 Backend

```bash
# Create S3 bucket for state
aws s3 mb s3://my-terraform-state-bucket-unique-name

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-terraform-state-bucket-unique-name \
  --versioning-configuration Status=Enabled

# Create DynamoDB table for locking
aws dynamodb create-table \
  --table-name terraform-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

### Configure Backend in Terraform

Create or update **providers.tf**:

```hcl
terraform {
  # Specify required providers
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }

  # Remote backend configuration
  backend "s3" {
    bucket         = "my-terraform-state-bucket-unique-name"
    key            = "eks-cluster/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}

# AWS Provider Configuration
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = "TWS-EKS-Tutorial"
      Environment = "Learning"
      ManagedBy   = "Terraform"
    }
  }
}
```

**Note:** For this tutorial, local backend is fine. Remote backend is a production best practice.

---

## Part 3: Creating VPC with Terraform

### Why Do We Need a VPC?

**VPC (Virtual Private Cloud)** provides:
- Network isolation
- Custom IP address range
- Public and private subnets
- Internet and NAT gateways
- Security through network segmentation

### VPC Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         VPC (10.0.0.0/16)                    │
│                                                               │
│  ┌─────────────────────────┐  ┌─────────────────────────┐  │
│  │   Public Subnet 1       │  │   Public Subnet 2       │  │
│  │   10.0.1.0/24           │  │   10.0.2.0/24           │  │
│  │   (AZ: us-west-2a)      │  │   (AZ: us-west-2b)      │  │
│  │                         │  │                         │  │
│  │   - NAT Gateway         │  │   - NAT Gateway         │  │
│  │   - Internet Gateway    │  │   - Load Balancers      │  │
│  └─────────────────────────┘  └─────────────────────────┘  │
│                                                               │
│  ┌─────────────────────────┐  ┌─────────────────────────┐  │
│  │   Private Subnet 1      │  │   Private Subnet 2      │  │
│  │   10.0.3.0/24           │  │   10.0.4.0/24           │  │
│  │   (AZ: us-west-2a)      │  │   (AZ: us-west-2b)      │  │
│  │                         │  │                         │  │
│  │   - EKS Nodes           │  │   - EKS Nodes           │  │
│  │   - Databases           │  │   - Databases           │  │
│  └─────────────────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### VPC Module Configuration

Create **modules/vpc/main.tf**:

```hcl
# Use AWS VPC Module (community-maintained)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = var.vpc_name
  cidr = var.vpc_cidr

  # Availability Zones
  azs = var.availability_zones

  # Subnets
  private_subnets = var.private_subnet_cidrs
  public_subnets  = var.public_subnet_cidrs

  # NAT Gateway - for private subnet internet access
  enable_nat_gateway     = true
  single_nat_gateway     = false  # One per AZ for high availability
  one_nat_gateway_per_az = true

  # DNS
  enable_dns_hostnames = true
  enable_dns_support   = true

  # Tags for EKS
  public_subnet_tags = {
    "kubernetes.io/role/elb" = "1"  # For external load balancers
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = "1"  # For internal load balancers
  }

  tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
  }
}
```

Create **modules/vpc/variables.tf**:

```hcl
variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "cluster_name" {
  description = "Name of EKS cluster"
  type        = string
}
```

Create **modules/vpc/outputs.tf**:

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = module.vpc.vpc_id
}

output "private_subnet_ids" {
  description = "IDs of private subnets"
  value       = module.vpc.private_subnets
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = module.vpc.public_subnets
}

output "nat_gateway_ids" {
  description = "IDs of NAT Gateways"
  value       = module.vpc.natgw_ids
}
```

---

## Part 4: Creating EKS Cluster with Auto Mode

### EKS Module Configuration

Create **modules/eks/main.tf**:

```hcl
# Use AWS EKS Module
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  # Enable EKS Auto Mode
  cluster_compute_config = {
    enabled    = true
    node_pools = ["general-purpose"]  # Auto Mode manages nodes
  }

  # VPC and Subnet Configuration
  vpc_id     = var.vpc_id
  subnet_ids = var.subnet_ids

  # Cluster endpoint access
  cluster_endpoint_public_access  = true   # Access from internet
  cluster_endpoint_private_access = true   # Access from VPC

  # Cluster Addons (managed by AWS)
  cluster_addons = {
    coredns = {
      most_recent = true
    }
    kube-proxy = {
      most_recent = true
    }
    vpc-cni = {
      most_recent = true
    }
    eks-pod-identity-agent = {
      most_recent = true
    }
  }

  # Enable IRSA (IAM Roles for Service Accounts)
  enable_irsa = true

  tags = var.tags
}

# IAM Role for Cluster
resource "aws_iam_role" "eks_cluster_role" {
  name = "${var.cluster_name}-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
    }]
  })
}

# Attach required policies
resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.eks_cluster_role.name
}

# IAM Role for Nodes (Auto Mode)
resource "aws_iam_role" "eks_node_role" {
  name = "${var.cluster_name}-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

# Attach node policies
resource "aws_iam_role_policy_attachment" "eks_node_policies" {
  for_each = toset([
    "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
    "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  ])

  policy_arn = each.value
  role       = aws_iam_role.eks_node_role.name
}
```

Create **modules/eks/variables.tf**:

```hcl
variable "cluster_name" {
  description = "Name of the EKS cluster"
  type        = string
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.31"
}

variable "vpc_id" {
  description = "VPC ID where EKS will be deployed"
  type        = string
}

variable "subnet_ids" {
  description = "List of subnet IDs for EKS"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

Create **modules/eks/outputs.tf**:

```hcl
output "cluster_id" {
  description = "EKS cluster ID"
  value       = module.eks.cluster_id
}

output "cluster_endpoint" {
  description = "Endpoint for EKS control plane"
  value       = module.eks.cluster_endpoint
}

output "cluster_name" {
  description = "EKS cluster name"
  value       = module.eks.cluster_name
}

output "cluster_certificate_authority_data" {
  description = "Base64 encoded certificate data"
  value       = module.eks.cluster_certificate_authority_data
  sensitive   = true
}

output "cluster_security_group_id" {
  description = "Security group ID attached to the EKS cluster"
  value       = module.eks.cluster_security_group_id
}
```

---

## Part 5: Main Terraform Configuration

### Root main.tf

Create **main.tf** in terraform directory:

```hcl
# Local variables
locals {
  cluster_name = "TWS-EKS-Cluster"
  region       = "us-west-2"
  
  tags = {
    Project     = "TWS-Kubernetes-Tutorial"
    Environment = "Learning"
    ManagedBy   = "Terraform"
  }
}

# VPC Module
module "vpc" {
  source = "./modules/vpc"

  vpc_name           = "${local.cluster_name}-vpc"
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["${local.region}a", "${local.region}b"]
  
  private_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnet_cidrs  = ["10.0.3.0/24", "10.0.4.0/24"]
  
  cluster_name = local.cluster_name
}

# EKS Module
module "eks" {
  source = "./modules/eks"

  cluster_name    = local.cluster_name
  cluster_version = "1.31"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = concat(module.vpc.private_subnet_ids, module.vpc.public_subnet_ids)
  
  tags = local.tags
  
  # Dependency
  depends_on = [module.vpc]
}
```

### Root variables.tf

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}
```

### Root outputs.tf

```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "eks_cluster_name" {
  description = "EKS Cluster Name"
  value       = module.eks.cluster_name
}

output "eks_cluster_endpoint" {
  description = "EKS Cluster Endpoint"
  value       = module.eks.cluster_endpoint
}

output "configure_kubectl" {
  description = "Command to configure kubectl"
  value       = "aws eks update-kubeconfig --region ${var.aws_region} --name ${module.eks.cluster_name}"
}
```

---

## Part 6: Deploying Infrastructure

### Step 1: Initialize Terraform

```bash
cd terraform
terraform init
```

**Expected Output:**
```
Initializing modules...
- eks in modules/eks
- vpc in modules/vpc

Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.x.x...

Terraform has been successfully initialized!
```

### Step 2: Create VPC First (Targeted Apply)

```bash
terraform apply -target=module.vpc
```

**Why target VPC first?**
- Creates network foundation
- Easier to troubleshoot
- Faster iteration
- Clear separation of concerns

**Review the plan:**
```
Terraform will perform the following actions:

  # module.vpc.module.vpc.aws_vpc.this[0] will be created
  + resource "aws_vpc" "this" {
      + cidr_block           = "10.0.0.0/16"
      + enable_dns_hostnames = true
      + enable_dns_support   = true
      ...
    }

  # 20+ more resources (subnets, route tables, NAT gateways, etc.)

Plan: 25 to add, 0 to change, 0 to destroy.
```

Type **yes** to create VPC.

**Creation time:** ~3-5 minutes

**What gets created:**
- 1 VPC
- 4 Subnets (2 public, 2 private)
- 2 NAT Gateways
- 1 Internet Gateway
- Route tables
- Security groups

### Step 3: Verify VPC Creation

```bash
# Check Terraform outputs
terraform output

# List VPCs in AWS
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=TWS-EKS-Cluster-vpc"

# List subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=<vpc-id>"
```

### Step 4: Deploy EKS Cluster

```bash
terraform apply -target=module.eks
```

**Review the plan:**
```
Terraform will perform the following actions:

  # module.eks.module.eks.aws_eks_cluster.this[0] will be created
  + resource "aws_eks_cluster" "this" {
      + name                      = "TWS-EKS-Cluster"
      + role_arn                  = (known after apply)
      + version                   = "1.31"
      ...
    }

Plan: 15 to add, 0 to change, 0 to destroy.
```

Type **yes** to create EKS cluster.

**Creation time:** ~8-10 minutes ☕

**What gets created:**
- EKS Control Plane
- IAM Roles and Policies
- Security Groups
- Cluster Add-ons
- Auto Mode Configuration

### Step 5: Monitor Creation

```bash
# In another terminal, watch cluster status
watch -n 5 'aws eks describe-cluster --name TWS-EKS-Cluster --query "cluster.status"'
```

**Status progression:**
```
"CREATING" → "ACTIVE"
```

---

## Part 7: Verifying Deployment

### Check Terraform State

```bash
# List all resources
terraform state list

# Show specific resource
terraform show module.eks
```

### Configure kubectl

```bash
# Update kubeconfig
aws eks update-kubeconfig --region us-west-2 --name TWS-EKS-Cluster

# Verify connection
kubectl cluster-info

# Check nodes (may be empty initially with Auto Mode)
kubectl get nodes
```

**Expected Output:**
```
Kubernetes control plane is running at https://xxxxx.eks.us-west-2.amazonaws.com
CoreDNS is running at https://xxxxx.eks.us-west-2.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

### Deploy Test Pod

```bash
# Create test namespace
kubectl create namespace test

# Deploy nginx
kubectl run nginx --image=nginx --namespace=test

# Wait for pod
kubectl wait --for=condition=Ready pod/nginx -n test --timeout=300s

# Check nodes (Auto Mode should have created one)
kubectl get nodes
```

**Success!** Auto Mode automatically created a node for your pod! 🎉

---

## Part 8: Understanding Terraform State

### What is State?

**terraform.tfstate** file contains:
- All created resources
- Resource metadata
- Dependencies
- Outputs

**Never manually edit state file!**

### State Commands

```bash
# List resources
terraform state list

# Show resource details
terraform state show module.vpc.module.vpc.aws_vpc.this[0]

# Remove resource from state (careful!)
terraform state rm module.eks

# Import existing resource
terraform import module.vpc.module.vpc.aws_vpc.this[0] vpc-xxxxx
```

---

## Part 9: Managing Infrastructure

### Making Changes

**Scenario:** Need to add tags

Edit **main.tf**:
```hcl
locals {
  tags = {
    Project     = "TWS-Kubernetes-Tutorial"
    Environment = "Learning"
    ManagedBy   = "Terraform"
    Owner       = "YourName"  # Added
  }
}
```

**Apply changes:**
```bash
terraform plan    # Preview changes
terraform apply   # Apply changes
```

### Updating Cluster Version

Edit **main.tf**:
```hcl
module "eks" {
  cluster_version = "1.32"  # Updated from 1.31
  ...
}
```

**Note:** Major version upgrades may require careful planning!

---

## Part 10: Cost Management

### Understanding Costs

**EKS Cluster:**
- Control plane: $0.10/hour (~$72/month)
- Nodes (Auto Mode): $0.05-$0.20/hour per node
- NAT Gateways: $0.045/hour per NAT (~$32/month each)

**Cost Optimization:**
1. **Delete when not using:**
   ```bash
   terraform destroy
   ```

2. **Use single NAT gateway for learning:**
   ```hcl
   single_nat_gateway = true
   ```

3. **Scale down nodes:**
   ```bash
   kubectl delete deployment --all
   # Auto Mode scales down automatically
   ```

---

## 🎓 Key Takeaways

### What You Learned
1. ✅ Terraform project structure and modules
2. ✅ Creating VPC with public/private subnets
3. ✅ Deploying EKS cluster with Auto Mode
4. ✅ Understanding IAM roles and policies
5. ✅ Managing Terraform state
6. ✅ Using targeted applies
7. ✅ Verifying deployments with kubectl

### Infrastructure Created

| Resource | Purpose | Cost |
|----------|---------|------|
| VPC | Network isolation | Free |
| Subnets | Network segmentation | Free |
| NAT Gateways | Private subnet internet | ~$32/mo each |
| EKS Control Plane | Kubernetes API | $72/mo |
| EKS Nodes (Auto) | Run workloads | Variable |

---

## 🧹 Cleanup

### Destroy Test Resources

```bash
# Delete test pod first
kubectl delete pod nginx -n test
kubectl delete namespace test

# Wait for nodes to scale down (Auto Mode)
# This may take a few minutes

# Destroy infrastructure
terraform destroy
```

**Destroy order:**
1. Kubernetes resources
2. EKS cluster
3. VPC and networking

**Total destroy time:** ~10-15 minutes

---

## 🚦 Next Steps

You've created production-grade infrastructure with Terraform! 

**Next Tutorial:** We'll deep dive into:
- Microservices architecture
- Helm charts structure
- Docker files for each service
- Service-to-service communication

---

## 📚 Quick Reference

### Essential Commands

```bash
# Terraform Workflow
terraform init
terraform fmt                    # Format code
terraform validate              # Validate syntax
terraform plan                  # Preview changes
terraform apply                 # Apply changes
terraform destroy               # Delete everything

# Targeted Operations
terraform apply -target=module.vpc
terraform destroy -target=module.eks

# State Management
terraform state list
terraform state show <resource>
terraform output

# kubectl Verification
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces
```

---

**Excellent work!** 🎉 Your infrastructure is deployed and ready for applications. Next, we'll deploy microservices! 🚀
