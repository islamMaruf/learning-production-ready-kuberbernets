# Tutorial 2: AWS CLI Setup & Understanding EKS Auto Mode

## 🎯 Learning Objectives
- Configure AWS CLI on your local machine
- Create IAM users with appropriate permissions
- Understand EKS (Elastic Kubernetes Service)
- Learn what makes EKS Auto Mode special
- Create your first EKS cluster manually (for learning)

## 📋 Prerequisites
- AWS account (new accounts get $100-$200 in credits)
- Terminal/Command line access
- Basic understanding of cloud concepts

---

## Part 1: AWS Account Setup

### Why AWS for Production Kubernetes?

When deploying Kubernetes in production, you need:
- **Reliability**: 99.95% uptime SLA
- **Managed Services**: Less operational overhead
- **Security**: Built-in security features
- **Scalability**: Auto-scaling capabilities
- **Support**: Enterprise support options

**Local Kubernetes (Kind, Minikube, K3s)** is great for:
- Learning
- Development
- Testing

**Cloud Kubernetes (EKS, GKE, AKS)** is required for:
- Production workloads
- Real users
- High availability
- Enterprise features

### AWS Free Tier & Credits

**For New Users:**
1. Create a new AWS account
2. Receive $100 initial credits automatically
3. Complete additional steps for up to $200 total credits
4. Perfect for running this project

**Important**: Always clean up resources after practice to avoid charges!

---

## Part 2: Installing & Configuring AWS CLI

### What is AWS CLI?

AWS Command Line Interface allows you to:
- Manage AWS services from terminal
- Automate infrastructure tasks
- Script deployments
- Configure EKS cluster access

### Installation

#### Check if Already Installed
```bash
aws --version
```

If installed, you'll see output like:
```
aws-cli/2.x.x Python/3.x.x ...
```

#### Install AWS CLI (if needed)

**For Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**For macOS:**
```bash
brew install awscli
```

**For Windows:**
Download from: https://aws.amazon.com/cli/

**Verify Installation:**
```bash
aws --version
```

---

## Part 3: Creating IAM User for CLI Access

### Understanding IAM (Identity and Access Management)

**Why IAM?**
- Root account is too powerful (full access)
- IAM users have specific permissions
- Follows security best practice: "Least Privilege"
- Easier to manage and audit access

### Step-by-Step: Create IAM User

#### Step 1: Access IAM Console
1. Log into AWS Console: https://console.aws.amazon.com
2. Search for "IAM" in the search bar
3. Click on "IAM" service

#### Step 2: Create New User
1. Click "Users" in left sidebar
2. Click "Create user" button
3. **User name**: `TWS-CLI-User` (or your preferred name)
4. **Uncheck** "Provide user access to AWS Management Console"
   - We only need CLI access, not console access
5. Click "Next"

#### Step 3: Assign Permissions
1. Select "Attach policies directly"
2. Search and select **"AdministratorAccess"**
   - For learning purposes, full access is convenient
   - In production, use more restrictive policies

3. Click "Next"
4. Review settings
5. Click "Create user"

#### Step 4: Generate Access Keys
1. Click on your newly created user (TWS-CLI-User)
2. Go to "Security credentials" tab
3. Scroll to "Access keys" section
4. Click "Create access key"
5. Select "Command Line Interface (CLI)"
6. Check the confirmation box
7. Click "Next"
8. (Optional) Add description tag
9. Click "Create access key"

**Important**: You'll see two values:
- **Access Key ID**: Like a username (you can see this later)
- **Secret Access Key**: Like a password (ONLY shown once!)

**Copy both values immediately!**

---

## Part 4: Configuring AWS CLI

### Configure AWS Credentials

Open your terminal and run:
```bash
aws configure
```

You'll be prompted for:

```
AWS Access Key ID [None]: <paste your Access Key ID>
AWS Secret Access Key [None]: <paste your Secret Access Key>
Default region name [None]: us-west-2
Default output format [None]: <press Enter>
```

**Region Explanation:**
- `us-west-2`: Oregon (recommended for this tutorial)
- `us-east-1`: Virginia (most services available)
- Choose closest to you for better latency
- Must use same region throughout tutorial

### Verify Configuration

Test your AWS CLI setup:
```bash
aws s3 ls
```

**Expected Output:**
- List of S3 buckets (might be empty if none created)
- If you see buckets or no error, configuration successful!

**If you see an error:**
- Check credentials are correct
- Verify IAM user has proper permissions
- Ensure region is valid

---

## Part 5: Understanding EKS (Elastic Kubernetes Service)

### What is EKS?

**Elastic Kubernetes Service** is AWS's managed Kubernetes offering.

### Kubernetes Architecture Review

#### Traditional Kubernetes Cluster Components:

**Control Plane** (The Brain):
- **API Server**: Handles all API requests
- **Scheduler**: Decides which node runs which pod
- **Controller Manager**: Maintains desired state
- **etcd**: Stores cluster configuration

**Worker Nodes** (The Workers):
- Run your application containers (pods)
- Managed by control plane
- Contain kubelet, container runtime

#### Standard EKS Behavior:

```
┌─────────────────────────────────┐
│      Control Plane              │
│  (AWS Manages This)             │
│  - API Server                   │
│  - Scheduler                    │
│  - Controller Manager           │
└─────────────────────────────────┘
            ↓
┌─────────────────────────────────┐
│      Data Plane                 │
│  (YOU Manage This)              │
│  - EC2 Instances (Nodes)        │
│  - EBS Volumes (Storage)        │
│  - VPC (Networking)             │
│  - Load Balancers               │
│  - CNI (Container Networking)   │
└─────────────────────────────────┘
```

**What YOU had to manage:**
- ✅ Control Plane: AWS manages
- ❌ EC2 Instances: You manage
- ❌ Storage (EBS): You manage
- ❌ Networking (VPC): You manage
- ❌ Load Balancers: You manage
- ❌ Security: You manage

---

## Part 6: Understanding EKS Auto Mode

### What is EKS Auto Mode? 🚀

**EKS Auto Mode** = AWS manages EVERYTHING!

```
┌─────────────────────────────────┐
│      Control Plane              │
│  ✅ AWS Manages                 │
└─────────────────────────────────┘
            ↓
┌─────────────────────────────────┐
│      Data Plane                 │
│  ✅ AWS Manages!                │
│  - EC2 Auto-scaling             │
│  - EBS Auto-provisioning        │
│  - VPC Auto-configuration       │
│  - Load Balancer Auto-create    │
│  - Security Auto-updates        │
└─────────────────────────────────┘
```

### Key Auto Mode Features

#### 1. **Automatic Node Management**
- **No manual EC2 management**: AWS creates/deletes nodes automatically
- **Carpenter Integration**: Auto-scaling based on workload
- **Cost Optimization**: Scales down to zero when idle

#### 2. **Enhanced Security**
- **Immutable AMIs**: Can't be modified after creation
- **21-Day Node Rotation**: Nodes automatically replaced for security
- **Automatic Security Patches**: Applied without downtime

#### 3. **Faster Cluster Creation**
- **Standard EKS**: 15-20 minutes
- **EKS Auto Mode**: Under 10 minutes
- Quick iteration for learning

#### 4. **Simplified Operations**
- No node groups to configure
- No launch templates needed
- No manual scaling policies
- Just deploy pods, AWS handles rest

### How Auto Mode Works

**When you deploy a pod:**
```
1. Pod created → 2. No node available? → 3. Auto Mode creates node
                                          ↓
4. Node ready ← 5. Pod scheduled ← 6. Application running
```

**When load decreases:**
```
1. Low utilization → 2. Auto Mode scales down → 3. Nodes terminated
                                                  ↓
4. Cost savings!
```

---

## Part 7: Creating EKS Cluster (Manual - For Learning)

### Quick Create with Auto Mode

Let's create a cluster manually to understand the process. Later, we'll use Terraform to automate this.

#### Step 1: Navigate to EKS Console
1. Go to AWS Console
2. Search for "EKS"
3. Click on "Elastic Kubernetes Service"

#### Step 2: Create Cluster
1. Click "Create cluster"
2. Select "Quick configuration with EKS Auto Mode"

#### Step 3: Configure Basic Settings
```
Cluster name: TWS-EKS-Cluster
Kubernetes version: 1.31 (latest)
```

#### Step 4: IAM Role Configuration
**Cluster IAM Role:**
- Click "Create recommended role"
- AWS creates role automatically
- Role name: `AmazonEKS_EKS_Cluster_Role`

**Node IAM Role:**
- Click "Create recommended role"
- AWS creates node role automatically
- Role name: `AmazonEKS_EKS_Node_Role`

#### Step 5: Networking
**VPC Settings:**
- Use default VPC (for learning)
- Or create new VPC (recommended for production)
- Subnets: Auto-selected in multiple AZs

**Security:**
- Default security groups applied
- Allows cluster-to-node communication

#### Step 6: Review & Create
1. Review all settings
2. Click "Create cluster"
3. Wait ~8-10 minutes for cluster to be active

**What's Happening:**
- AWS creates control plane
- Sets up networking
- Configures security groups
- Enables Auto Mode
- Makes cluster ready for workloads

---

## Part 8: Understanding Cluster Creation Output

### Once Cluster is Active

**You'll see:**
- ✅ Cluster Status: Active
- ✅ Kubernetes Version: 1.31
- ✅ Cluster Endpoint: https://xxxxx.eks.region.amazonaws.com
- ✅ Auto Mode: Enabled

**Important Note:**
- ⚠️ No nodes yet! (This is expected with Auto Mode)
- Nodes only created when you deploy pods
- This saves money when cluster is idle

---

## Part 9: Verifying Cluster Access

### Connect kubectl to Your Cluster

**Update kubeconfig:**
```bash
aws eks update-kubeconfig \
  --region us-west-2 \
  --name TWS-EKS-Cluster
```

**What this does:**
- Downloads cluster certificate
- Adds cluster to kubectl config
- Configures authentication

**Verify connection:**
```bash
kubectl get nodes
```

**Expected Output:**
```
No resources found
```

**This is correct!** Auto Mode hasn't created nodes yet because no pods are deployed.

### Test by Deploying a Pod

```bash
# Create a namespace
kubectl create namespace demo

# Deploy nginx
kubectl run nginx --image=nginx --namespace=demo

# Check nodes (after a moment)
kubectl get nodes
```

**Now you'll see:**
```
NAME                         STATUS   ROLE    AGE
ip-10-0-1-234.ec2.internal   Ready    node    45s
```

**Auto Mode created a node automatically!** 🎉

---

## 🎓 Key Takeaways

### What You Learned
1. ✅ AWS CLI installation and configuration
2. ✅ IAM user creation with proper permissions
3. ✅ Understanding EKS architecture
4. ✅ Key differences between standard EKS and Auto Mode
5. ✅ Creating an EKS cluster manually
6. ✅ Connecting kubectl to EKS cluster

### EKS vs EKS Auto Mode

| Feature | Standard EKS | EKS Auto Mode |
|---------|-------------|---------------|
| Control Plane | AWS Manages | AWS Manages |
| Node Management | You Manage | AWS Manages |
| Storage | You Manage | AWS Manages |
| Networking | You Manage | AWS Manages |
| Security Updates | You Manage | AWS Manages |
| Cost | Pay for unused nodes | Pay only for what's used |
| Setup Time | 15-20 min | Under 10 min |

### Cost Comparison

**Standard EKS:**
- Fixed node costs even when idle
- Must over-provision for peak load
- Manual scaling required

**EKS Auto Mode:**
- Pay only for actual usage
- Auto-scales to zero when idle
- Automatic right-sizing
- ~30-40% cost savings typical

---

## 🧹 Cleanup (Important!)

### Delete Manual Test Cluster

**To avoid charges:**
```bash
# List clusters
aws eks list-clusters

# Delete cluster (in AWS Console)
1. Go to EKS Console
2. Select TWS-EKS-Cluster
3. Click Delete
4. Type cluster name to confirm
5. Wait for deletion
```

**Why delete?**
- EKS control plane costs $0.10/hour
- Nodes (if any) cost extra
- We'll recreate with Terraform in next tutorial

---

## 🚦 Next Steps

You now understand:
- How to configure AWS CLI
- What EKS Auto Mode provides
- How to create clusters manually

**Next Tutorial:** We'll install all required tools (Terraform, kubectl, Helm) and set up the complete development environment.

---

## 📚 Additional Resources

- [AWS EKS Auto Mode Documentation](https://docs.aws.amazon.com/eks/latest/userguide/automode.html)
- [AWS CLI Configuration Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

**Great job completing Tutorial 2!** 🎉 Ready for Tutorial 3? Let's install all the tools! 🚀
