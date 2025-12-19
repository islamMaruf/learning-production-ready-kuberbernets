# Tutorial 1: Introduction & Project Overview

## 🎯 What You'll Learn
In this tutorial, you'll understand the complete architecture of a production-ready Kubernetes deployment using AWS EKS Auto Mode, Terraform, ArgoCD, and GitOps practices.

## 📋 Prerequisites for This Tutorial Series
Before starting, you should have basic knowledge of:
- Docker (containerization basics)
- Kubernetes (pods, deployments, services)
- Linux command line
- Git version control
- Terraform basics (infrastructure as code)

> **Note**: This is a hands-on project tutorial, not a basics course. If you need to learn the fundamentals, complete the "DevOps in One Shot" series first.

## 🏗️ Project Overview

### What We're Building
A **multi-tier e-commerce retail store application** called "The Most Public Secret Shop" - a fully functional online store deployed on production-ready Kubernetes infrastructure.

### Application Architecture

#### Frontend
- **UI Service**: Built with Java (Spring Boot)
  - Handles user interface
  - Product catalog browsing
  - Shopping cart interface

#### Backend Microservices
1. **Catalog Service** (Golang)
   - Manages product catalog
   - Product information API
   - Connected to MySQL database

2. **Checkout Service** (Node.js)
   - Handles checkout process
   - Uses Redis for caching
   - Temporary session data

3. **Orders Service** (Java)
   - Order management
   - Order history
   - Connected to PostgreSQL

4. **Cart Service** (Java)
   - Shopping cart operations
   - Connected to DynamoDB

#### Databases & Caching
- **Redis**: Caching layer for checkout service
- **PostgreSQL**: Orders database
- **DynamoDB**: Cart data storage
- **MySQL**: Product catalog database

### Why Microservices Architecture?

**Advantages of Microservices:**
1. **Independent Scaling**: Scale individual services based on demand
2. **Fault Isolation**: If orders service fails, catalog and UI continue working
3. **Technology Flexibility**: Use different languages for different services
4. **Independent Deployment**: Update services without affecting others
5. **Easier Maintenance**: Smaller, focused codebases

**Comparison with Monolithic Architecture:**
- **Monolithic**: One large application, all components together
  - Single point of failure
  - Must scale entire application
  - Technology locked to one stack

- **Microservices**: Multiple small services, independently deployable
  - Isolated failures
  - Scale only what's needed
  - Best for Kubernetes orchestration

## 🚀 Infrastructure Stack

### Core Technologies

#### 1. AWS EKS Auto Mode
**What is it?**
- Elastic Kubernetes Service - AWS's managed Kubernetes
- **Auto Mode**: AWS manages BOTH control plane AND data plane
- Production-ready in under 10 minutes

**What AWS Manages:**
- **Control Plane**: API Server, Scheduler, Controller Manager
- **Data Plane**: EC2 instances (nodes), Storage (EBS volumes), Networking (VPC, CNI), Load Balancers
- **Security**: Immutable AMIs, 21-day node rotation

#### 2. Terraform
- Infrastructure as Code (IaC) tool
- Automates infrastructure provisioning
- Version-controlled infrastructure
- Reusable modules for VPC and EKS

#### 3. ArgoCD
- GitOps Continuous Delivery tool
- Automatically syncs applications from Git to Kubernetes
- Declarative application deployment
- Self-healing capabilities

#### 4. Helm
- Kubernetes package manager
- Templated manifests with values
- Easy application configuration
- Reusable charts for all services

#### 5. Nginx Ingress Controller
- Routes external traffic to internal services
- Load balancing
- SSL/TLS termination support

## 📊 Project Phases

### Phase 1: Architecture & Infrastructure Setup
**Objectives:**
- Understand project architecture
- Set up AWS CLI and configure credentials
- Install required tools (Terraform, kubectl, Helm)
- Create VPC using Terraform
- Deploy EKS cluster with Auto Mode
- Configure kubectl access

**Deliverables:**
- Running EKS cluster
- Configured VPC with proper networking
- Local tools configured and connected

### Phase 2: Kubernetes Application Deployment
**Objectives:**
- Deep dive into microservices architecture
- Understand Helm chart structure
- Review Docker files for each service
- Deploy applications using ArgoCD
- Configure service-to-service communication
- Set up Ingress for external access

**Deliverables:**
- All microservices deployed and running
- ArgoCD managing applications
- Functional e-commerce website

### Phase 3: GitOps & CI/CD Pipeline
**Objectives:**
- Implement GitHub Actions workflows
- Set up ECR (Elastic Container Registry) for private images
- Create automated build and push pipeline
- Update Helm values automatically
- Implement zero-touch deployment
- Test complete GitOps workflow

**Deliverables:**
- Automated CI/CD pipeline
- Private container images in ECR
- Git as single source of truth
- Working GitOps deployment

## 🎓 Learning Outcomes

By completing this tutorial, you will:

### Technical Skills
1. **Infrastructure as Code**: Create production infrastructure with Terraform
2. **Kubernetes Cluster Management**: Deploy and manage EKS Auto Mode clusters
3. **Container Orchestration**: Understand pod scheduling, scaling, and networking
4. **GitOps Methodology**: Implement declarative configuration management
5. **CI/CD Pipelines**: Build automated deployment workflows
6. **AWS Cloud Services**: Work with EKS, VPC, ECR, IAM, and more

### DevOps Practices
1. **Version Control**: Infrastructure and application code in Git
2. **Monitoring & Observability**: Track application health
3. **Security Best Practices**: IAM roles, least privilege access, immutable infrastructure
4. **Troubleshooting**: Debug deployment issues, access problems, connectivity

### Modern Architecture Patterns
1. **Microservices Design**: Service boundaries, data flow, inter-service communication
2. **Deployment Strategies**: Rolling updates, blue-green, canary deployments
3. **GitOps as Deployment Method**: Git as single source of truth
4. **Production Kubernetes**: Real-world EKS management
5. **End-to-End DevOps Workflow**: From code commit to production deployment

## 💰 Cost Considerations

### AWS Free Tier & Credits
- **New AWS Accounts**: Get $100-$200 in credits
- **EKS Cluster**: ~$0.10/hour for control plane
- **Auto Mode Benefits**: Pay only for resources used
- **Cost Optimization**: Cluster auto-scales down when idle

### Cleanup
Always remember to destroy resources when done:
```bash
terraform destroy
```

## 🎯 Success Criteria

You'll know you're successful when:
- ✅ EKS cluster is running with Auto Mode enabled
- ✅ All microservices are deployed and healthy
- ✅ You can browse the e-commerce website
- ✅ You can complete a purchase flow (add to cart → checkout → order)
- ✅ ArgoCD is syncing applications from Git
- ✅ GitHub Actions automatically build and deploy changes
- ✅ You understand the complete architecture

## 📚 Project Repository Structure

```
retail-store-sample-app/
├── .github/
│   └── workflows/          # GitHub Actions CI/CD pipelines
├── argocd/
│   ├── applications/       # ArgoCD application definitions
│   └── projects/           # ArgoCD project configurations
├── src/
│   ├── ui/                 # Java Spring Boot frontend
│   │   ├── charts/         # Helm charts for UI
│   │   └── Dockerfile      # Container build instructions
│   ├── cart/               # Java cart service
│   ├── catalog/            # Golang catalog service
│   ├── checkout/           # Node.js checkout service
│   └── orders/             # Java orders service
├── terraform/              # Infrastructure as Code
│   ├── main.tf            # Main Terraform configuration
│   ├── variables.tf       # Configurable variables
│   ├── vpc.tf             # VPC module
│   └── eks.tf             # EKS cluster module
└── README.md              # Project documentation
```

## 🔄 Data Flow Architecture

```
User → Load Balancer → Nginx Ingress → UI Service
                                          ↓
                                    [Catalog API]
                                          ↓
                                    MySQL Database

User adds to cart → Cart Service → DynamoDB
User checks out → Checkout Service → Redis Cache
User places order → Orders Service → PostgreSQL
```

## 🛡️ Security Features

1. **IAM Roles**: Least privilege access for services
2. **VPC Isolation**: Private subnets for databases
3. **Immutable Infrastructure**: Nodes replaced regularly
4. **Secret Management**: Kubernetes secrets for sensitive data
5. **Network Policies**: Control inter-service communication

## 🚦 Next Steps

Now that you understand the project architecture, proceed to:
- **Tutorial 2**: AWS CLI Setup & EKS Auto Mode Configuration

---

## 📝 Quick Reference

### Key Commands You'll Use
```bash
# AWS CLI
aws configure
aws eks update-kubeconfig

# Terraform
terraform init
terraform plan
terraform apply
terraform destroy

# Kubectl
kubectl get pods
kubectl get services
kubectl logs <pod-name>

# Helm
helm version
helm list
helm install

# ArgoCD
argocd app list
argocd app sync
```

### Important URLs
- **AWS Console**: https://console.aws.amazon.com
- **EKS Documentation**: https://docs.aws.amazon.com/eks/
- **ArgoCD Docs**: https://argo-cd.readthedocs.io/
- **Terraform Registry**: https://registry.terraform.io/

---

**Ready to begin?** Head to Tutorial 2 to start setting up your environment! 🚀
