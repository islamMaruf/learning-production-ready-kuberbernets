# Tutorial 3: Prerequisites & Tool Installation

## 🎯 Learning Objectives
- Install and verify all required tools for the project
- Configure your development environment
- Understand what each tool does and why it's needed
- Set up your IDE with helpful extensions
- Prepare for infrastructure deployment

## 📋 Required Tools Overview

| Tool | Purpose | Version |
|------|---------|---------|
| **AWS CLI** | Manage AWS services | 2.x |
| **Terraform** | Infrastructure as Code | 1.5+ |
| **kubectl** | Kubernetes command-line tool | 1.31+ |
| **Helm** | Kubernetes package manager | 3.x |
| **Git** | Version control | 2.x |
| **Docker** | Container runtime (optional) | 24.x |

---

## Part 1: Verifying AWS CLI Installation

### Check AWS CLI

From Tutorial 2, you should already have AWS CLI installed.

```bash
aws --version
```

**Expected Output:**
```
aws-cli/2.15.0 Python/3.11.x ...
```

### Verify AWS Configuration

```bash
# Check configured credentials
aws sts get-caller-identity
```

**Expected Output:**
```json
{
    "UserId": "AIDAXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/TWS-CLI-User"
}
```

✅ If you see your account details, AWS CLI is properly configured!

---

## Part 2: Installing Terraform

### What is Terraform?

**Terraform** is an Infrastructure as Code (IaC) tool that allows you to:
- Define infrastructure in code files
- Version control your infrastructure
- Automate resource creation
- Ensure consistency across environments
- Preview changes before applying

### Why Use Terraform?

**Without Terraform (Manual):**
```
1. Log into AWS Console
2. Click through 20+ settings
3. Manually configure VPC
4. Manually create subnets
5. Manually set up routing
6. Manually create EKS cluster
7. Hope you didn't miss anything!
```

**With Terraform:**
```bash
terraform apply
# Everything created automatically in minutes!
```

### Installation Steps

#### For Linux (Ubuntu/Debian)
```bash
# Add HashiCorp GPG key
wget -O- https://apt.releases.hashicorp.com/gpg | \
    sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Add HashiCorp repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
    https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
    sudo tee /etc/apt/sources.list.d/hashicorp.list

# Update and install
sudo apt update
sudo apt install terraform
```

#### For macOS
```bash
# Using Homebrew
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

#### For Windows
1. Download from: https://www.terraform.io/downloads
2. Extract the ZIP file
3. Add terraform.exe to your PATH

### Verify Installation

```bash
terraform version
```

**Expected Output:**
```
Terraform v1.7.0
on linux_amd64
```

### Understanding Terraform Basics

#### Main Commands You'll Use:

**1. Initialize**
```bash
terraform init
```
- Downloads provider plugins (AWS, Kubernetes, etc.)
- Initializes backend
- Prepares working directory

**2. Plan**
```bash
terraform plan
```
- Shows what will be created/modified/destroyed
- Like a "dry run"
- No actual changes made

**3. Apply**
```bash
terraform apply
```
- Actually creates/modifies infrastructure
- Prompts for confirmation
- Saves state file

**4. Destroy**
```bash
terraform destroy
```
- Deletes all managed infrastructure
- Use for cleanup
- Saves money!

---

## Part 3: Installing kubectl

### What is kubectl?

**kubectl** (Kubernetes Control) is the command-line tool to:
- Interact with Kubernetes clusters
- Deploy applications
- Inspect resources
- View logs
- Troubleshoot issues

### Installation Steps

#### For Linux
```bash
# Download latest version
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

#### For macOS
```bash
# Using Homebrew
brew install kubectl

# Or using curl (same as Linux)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### For Windows
```powershell
# Using Chocolatey
choco install kubernetes-cli

# Or download from:
# https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
```

### Verify Installation

```bash
kubectl version --client
```

**Expected Output:**
```
Client Version: v1.31.0
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

### Essential kubectl Commands

```bash
# View cluster info
kubectl cluster-info

# List all pods
kubectl get pods

# List all pods across all namespaces
kubectl get pods --all-namespaces

# List nodes
kubectl get nodes

# Describe a resource
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Apply configuration
kubectl apply -f file.yaml

# Delete resource
kubectl delete pod <pod-name>
```

---

## Part 4: Installing Helm

### What is Helm?

**Helm** is the package manager for Kubernetes (think: apt/yum for K8s)

**Benefits:**
1. **Templating**: Reusable Kubernetes manifests
2. **Versioning**: Roll back deployments easily
3. **Packaging**: Bundle related resources together
4. **Configuration Management**: Override default values

### Helm Concepts

**Chart**: A package of Kubernetes resources
```
my-app-chart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration
└── templates/          # Kubernetes manifest templates
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

**Values**: Configuration parameters you can override
```yaml
# values.yaml
replicas: 3
image:
  repository: myapp
  tag: v1.0.0
```

**Release**: An instance of a chart running in cluster

### Installation Steps

#### For Linux
```bash
# Download installation script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

#### For macOS
```bash
# Using Homebrew
brew install helm

# Verify
helm version
```

#### For Windows
```powershell
# Using Chocolatey
choco install kubernetes-helm

# Or using Scoop
scoop install helm
```

### Verify Installation

```bash
helm version
```

**Expected Output:**
```
version.BuildInfo{Version:"v3.14.0", GitCommit:"...", GitTreeState:"clean", GoVersion:"go1.21.5"}
```

### Essential Helm Commands

```bash
# Search for charts
helm search repo nginx

# Install a chart
helm install my-release chart-name

# List releases
helm list

# Upgrade a release
helm upgrade my-release chart-name

# Rollback a release
helm rollback my-release 1

# Uninstall a release
helm uninstall my-release

# Show chart values
helm show values chart-name
```

---

## Part 5: Installing Git

### What is Git?

Version control system for:
- Tracking code changes
- Collaboration
- GitOps workflows
- Infrastructure as Code versioning

### Installation

#### For Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

#### For macOS
```bash
brew install git
```

#### For Windows
Download from: https://git-scm.com/download/win

### Configure Git

```bash
# Set your name
git config --global user.name "Your Name"

# Set your email
git config --global user.email "your.email@example.com"

# Verify configuration
git config --list
```

### Verify Installation

```bash
git --version
```

**Expected Output:**
```
git version 2.43.0
```

---

## Part 6: Optional - Installing Docker

### Why Docker?

While not strictly required for this tutorial (we'll use pre-built images), Docker helps you:
- Build custom container images
- Test locally before deploying
- Understand containerization better

### Installation

#### For Linux (Ubuntu)
```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc

# Install dependencies
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release

# Add Docker's GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin

# Add user to docker group (no sudo needed)
sudo usermod -aG docker $USER

# Log out and back in for group changes to take effect
```

#### For macOS
```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# Or using Homebrew
brew install --cask docker
```

#### For Windows
Download Docker Desktop from: https://www.docker.com/products/docker-desktop

### Verify Installation

```bash
docker --version
docker ps
```

**Expected Output:**
```
Docker version 25.0.0, build 4584deb
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

---

## Part 7: IDE Setup (Recommended)

### VS Code Extensions

If using Visual Studio Code, install these extensions:

```
1. HashiCorp Terraform
   - Syntax highlighting
   - Auto-completion
   - Validation

2. Kubernetes
   - Cluster management
   - YAML validation
   - Resource visualization

3. YAML
   - YAML syntax support
   - Schema validation
   - Formatting

4. Docker
   - Container management
   - Dockerfile support
   - Image building

5. GitLens
   - Git visualization
   - Blame annotations
   - History tracking
```

### Cursor IDE with Amazon Q (Alternative)

**Amazon Q** is an AI coding assistant:
- Integrated in Cursor IDE
- Helps debug Terraform errors
- Suggests Kubernetes configurations
- Explains AWS services
- Free for personal use

**Setup:**
1. Download Cursor IDE: https://cursor.sh/
2. Install Amazon Q extension
3. Sign in with Builder ID (free AWS account)

---

## Part 8: Setting Up Project Repository

### Clone the Tutorial Repository

```bash
# Create workspace directory
mkdir -p ~/workspace/kubernetes-projects
cd ~/workspace/kubernetes-projects

# Clone the repository
git clone https://github.com/aws-samples/eks-workshop-sample-api-service-go.git retail-store-app

cd retail-store-app
```

**Repository Structure:**
```
retail-store-app/
├── terraform/              # Infrastructure code
│   ├── main.tf
│   ├── variables.tf
│   └── modules/
├── src/                    # Application source code
│   ├── ui/
│   ├── cart/
│   ├── catalog/
│   ├── checkout/
│   └── orders/
├── helm-charts/            # Helm charts for services
└── argocd/                 # ArgoCD configurations
```

### Initialize Terraform

```bash
cd terraform
terraform init
```

**What happens:**
```
Initializing modules...
Initializing provider plugins...
- Installing hashicorp/aws v5.x.x...
- Installing hashicorp/kubernetes v2.x.x...

Terraform has been successfully initialized!
```

---

## Part 9: Verification Checklist

### Run All Version Checks

Create a verification script:

```bash
#!/bin/bash
# save as check-tools.sh

echo "🔍 Checking installed tools..."
echo ""

# AWS CLI
echo "✅ AWS CLI:"
aws --version
echo ""

# Terraform
echo "✅ Terraform:"
terraform version | head -n 1
echo ""

# kubectl
echo "✅ kubectl:"
kubectl version --client --short 2>/dev/null || kubectl version --client
echo ""

# Helm
echo "✅ Helm:"
helm version --short
echo ""

# Git
echo "✅ Git:"
git --version
echo ""

# Docker (optional)
echo "✅ Docker (optional):"
docker --version 2>/dev/null || echo "Not installed (optional)"
echo ""

# AWS Configuration
echo "✅ AWS Configuration:"
aws sts get-caller-identity 2>/dev/null && echo "Configured correctly!" || echo "Not configured"
echo ""

echo "🎉 Tool check complete!"
```

**Make executable and run:**
```bash
chmod +x check-tools.sh
./check-tools.sh
```

### Expected Output Summary

```
✅ AWS CLI: aws-cli/2.15.0
✅ Terraform: Terraform v1.7.0
✅ kubectl: Client Version: v1.31.0
✅ Helm: v3.14.0
✅ Git: git version 2.43.0
✅ Docker: Docker version 25.0.0
✅ AWS Configuration: Configured correctly!
```

---

## Part 10: Understanding the Tool Chain

### How Tools Work Together

```
Developer
    ↓
[Git Repository]
    ↓
[Terraform] → Creates Infrastructure
    ↓
[AWS EKS Cluster]
    ↓
[kubectl] → Manages Cluster
    ↓
[Helm] → Deploys Applications
    ↓
[ArgoCD] → GitOps Sync
    ↓
[Running Application]
```

### Workflow Example

**1. Infrastructure Creation (Terraform)**
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

**2. Connect to Cluster (kubectl)**
```bash
aws eks update-kubeconfig --name my-cluster --region us-west-2
kubectl get nodes
```

**3. Deploy Application (Helm)**
```bash
helm install my-app ./helm-charts/my-app
kubectl get pods
```

**4. GitOps Automation (ArgoCD)**
```bash
# ArgoCD automatically syncs from Git
# No manual commands needed!
```

---

## 🎓 Key Takeaways

### What You Learned
1. ✅ Installed all required tools (AWS CLI, Terraform, kubectl, Helm)
2. ✅ Verified each tool is working correctly
3. ✅ Understood what each tool does
4. ✅ Set up development environment
5. ✅ Cloned project repository
6. ✅ Initialized Terraform

### Tool Summary

| Tool | When to Use | Command Example |
|------|------------|-----------------|
| AWS CLI | AWS account operations | `aws s3 ls` |
| Terraform | Infrastructure creation | `terraform apply` |
| kubectl | Kubernetes management | `kubectl get pods` |
| Helm | App deployment | `helm install app` |
| Git | Version control | `git commit -m "msg"` |

---

## 🧹 Troubleshooting

### Common Issues

#### Issue: "command not found"
**Solution:**
```bash
# Check if installed
which terraform
which kubectl

# If not found, add to PATH
export PATH=$PATH:/usr/local/bin

# Make permanent (add to ~/.bashrc or ~/.zshrc)
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

#### Issue: AWS CLI not configured
**Solution:**
```bash
aws configure
# Enter Access Key ID, Secret Key, Region
```

#### Issue: kubectl can't connect
**Solution:**
```bash
# Update kubeconfig
aws eks update-kubeconfig --name cluster-name --region us-west-2

# Check connection
kubectl cluster-info
```

#### Issue: Permission denied (Docker)
**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker
```

---

## 🚦 Next Steps

You now have all tools installed and verified! 

**Next Tutorial:** We'll use Terraform to create the complete infrastructure:
- VPC with public/private subnets
- EKS cluster with Auto Mode
- Security groups and IAM roles
- Network configuration

Everything will be automated - no clicking in AWS Console!

---

## 📚 Quick Reference

### Essential Commands

```bash
# Check tool versions
aws --version
terraform version
kubectl version --client
helm version
git --version

# AWS Configuration
aws configure
aws sts get-caller-identity

# Terraform Workflow
terraform init
terraform plan
terraform apply
terraform destroy

# Kubernetes Operations
kubectl get pods
kubectl get services
kubectl logs <pod>
kubectl describe pod <pod>

# Helm Operations
helm list
helm install <name> <chart>
helm upgrade <name> <chart>
helm uninstall <name>
```

---

**Congratulations!** 🎉 Your development environment is ready. Let's build infrastructure in Tutorial 4! 🚀
