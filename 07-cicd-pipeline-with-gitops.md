# Tutorial 7: Complete CI/CD Pipeline with GitOps

## 🎯 Learning Objectives
- Build complete CI/CD pipeline with GitHub Actions
- Automate Docker image building and pushing to ECR
- Implement automated Helm value updates
- Integrate GitHub Actions with ArgoCD
- Implement zero-touch deployments
- Set up environment-specific pipelines

## 📋 Prerequisites
- Completed Tutorial 6 (ArgoCD configured)
- GitHub account with repository access
- AWS ECR repository created
- ArgoCD applications deployed

---

## Part 1: Understanding CI/CD Pipeline

### Complete Pipeline Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTINUOUS INTEGRATION                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Developer Commits Code                                      │
│         ↓                                                    │
│  GitHub Actions Triggered                                    │
│         ↓                                                    │
│  Run Tests & Linting                                         │
│         ↓                                                    │
│  Build Docker Image                                          │
│         ↓                                                    │
│  Push to AWS ECR                                             │
│         ↓                                                    │
│  Update Helm values.yaml (new image tag)                     │
│         ↓                                                    │
│  Commit values.yaml change                                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                  CONTINUOUS DEPLOYMENT                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ArgoCD Detects Git Change                                   │
│         ↓                                                    │
│  Syncs New Config to Cluster                                 │
│         ↓                                                    │
│  Kubernetes Rolling Update                                   │
│         ↓                                                    │
│  Application Running with New Version                        │
└─────────────────────────────────────────────────────────────┘
```

### Pipeline Stages

**Stage 1: Code Push**
- Developer commits code
- Push to GitHub repository
- Triggers GitHub Actions

**Stage 2: Build & Test**
- Checkout code
- Run unit tests
- Lint code
- Build Docker image

**Stage 3: Push to Registry**
- Authenticate with AWS ECR
- Tag image with version
- Push image to ECR

**Stage 4: Update Manifests**
- Update Helm values.yaml
- Commit new image tag
- Push to Git repository

**Stage 5: Deploy (ArgoCD)**
- ArgoCD watches Git
- Detects change
- Syncs to cluster
- Rolling update

---

## Part 2: Setting Up AWS ECR

### Create ECR Repositories

**For each service:**

```bash
# Create repositories
aws ecr create-repository --repository-name retail-store/ui --region us-west-2
aws ecr create-repository --repository-name retail-store/catalog --region us-west-2
aws ecr create-repository --repository-name retail-store/cart --region us-west-2
aws ecr create-repository --repository-name retail-store/checkout --region us-west-2
aws ecr create-repository --repository-name retail-store/orders --region us-west-2
```

**Using Terraform:**

```hcl
# ECR repositories
resource "aws_ecr_repository" "services" {
  for_each = toset(["ui", "catalog", "cart", "checkout", "orders"])
  
  name                 = "retail-store/${each.key}"
  image_tag_mutability = "MUTABLE"

  image_scanning_configuration {
    scan_on_push = true  # Scan images for vulnerabilities
  }

  encryption_configuration {
    encryption_type = "AES256"
  }

  tags = {
    Service = each.key
  }
}

# Lifecycle policy to clean old images
resource "aws_ecr_lifecycle_policy" "services" {
  for_each = aws_ecr_repository.services

  repository = each.value.name

  policy = jsonencode({
    rules = [{
      rulePriority = 1
      description  = "Keep last 10 images"
      selection = {
        tagStatus     = "tagged"
        tagPrefixList = ["v"]
        countType     = "imageCountMoreThan"
        countNumber   = 10
      }
      action = {
        type = "expire"
      }
    }]
  })
}

# Output ECR URLs
output "ecr_urls" {
  value = {
    for k, v in aws_ecr_repository.services :
    k => v.repository_url
  }
}
```

Apply:
```bash
terraform apply -target=aws_ecr_repository.services
```

### Get ECR Login Command

```bash
# Get login password
aws ecr get-login-password --region us-west-2 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-west-2.amazonaws.com
```

---

## Part 3: GitHub Actions Basics

### Repository Structure for CI/CD

```
retail-store-app/
├── .github/
│   └── workflows/
│       ├── ui-service-ci.yml
│       ├── catalog-service-ci.yml
│       ├── cart-service-ci.yml
│       ├── checkout-service-ci.yml
│       └── orders-service-ci.yml
├── src/
│   ├── ui/
│   │   ├── Dockerfile
│   │   ├── pom.xml
│   │   └── src/
│   ├── catalog/
│   │   ├── Dockerfile
│   │   ├── go.mod
│   │   └── main.go
│   └── ...
├── helm-charts/
│   ├── ui/
│   │   └── values.yaml
│   └── ...
└── README.md
```

### GitHub Actions Concepts

**Workflow**: YAML file defining automation
**Job**: Group of steps
**Step**: Individual task
**Runner**: Server executing workflow
**Action**: Reusable unit of code
**Secret**: Encrypted credential

---

## Part 4: Creating GitHub Actions Workflows

### UI Service CI/CD Pipeline

Create `.github/workflows/ui-service-ci.yml`:

```yaml
name: UI Service CI/CD

# Trigger on push to main or tags
on:
  push:
    branches:
      - main
    tags:
      - 'ui-v*'
    paths:
      - 'src/ui/**'
      - 'helm-charts/ui/**'
      - '.github/workflows/ui-service-ci.yml'
  pull_request:
    branches:
      - main
    paths:
      - 'src/ui/**'

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: retail-store/ui
  SERVICE_NAME: ui

jobs:
  # Job 1: Build and Test
  build-test:
    name: Build and Test
    runs-on: ubuntu-latest
    
    steps:
      # Checkout code
      - name: Checkout code
        uses: actions/checkout@v4
      
      # Set up Java
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      
      # Run tests
      - name: Run unit tests
        run: |
          cd src/ui
          mvn test
      
      # Build application
      - name: Build application
        run: |
          cd src/ui
          mvn package -DskipTests
      
      # Upload artifact for next job
      - name: Upload build artifact
        uses: actions/upload-artifact@v3
        with:
          name: ui-jar
          path: src/ui/target/*.jar
  
  # Job 2: Build and Push Docker Image
  build-push-image:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: build-test  # Run after build-test
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/')
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      # Configure AWS credentials
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      # Login to Amazon ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      # Docker meta (tags and labels)
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}
      
      # Set up Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # Build and push
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./src/ui
          file: ./src/ui/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      # Scan image for vulnerabilities
      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:latest
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      # Upload scan results
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
  
  # Job 3: Update Helm Values
  update-helm-values:
    name: Update Helm Values
    runs-on: ubuntu-latest
    needs: build-push-image
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
      
      # Extract image tag
      - name: Get image tag
        id: image-tag
        run: |
          TAG=$(echo "${{ needs.build-push-image.outputs.image-tag }}" | grep -oP 'sha-\w+' | head -1)
          echo "tag=$TAG" >> $GITHUB_OUTPUT
      
      # Update values.yaml
      - name: Update Helm values
        run: |
          cd helm-charts/ui
          
          # Update image tag in values.yaml
          sed -i "s/tag: .*/tag: \"${{ steps.image-tag.outputs.tag }}\"/" values.yaml
          
          # Also update in values-prod.yaml if exists
          if [ -f values-prod.yaml ]; then
            sed -i "s/tag: .*/tag: \"${{ steps.image-tag.outputs.tag }}\"/" values-prod.yaml
          fi
      
      # Commit and push changes
      - name: Commit changes
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add helm-charts/ui/values*.yaml
          git commit -m "Update UI image tag to ${{ steps.image-tag.outputs.tag }}"
          git push
```

### Key Workflow Components Explained

**Trigger Configuration:**
```yaml
on:
  push:
    branches: [main]
    paths: ['src/ui/**']  # Only run if UI code changes
```

**Job Dependencies:**
```yaml
jobs:
  build:
    ...
  deploy:
    needs: build  # Run after build succeeds
```

**Conditional Execution:**
```yaml
if: github.ref == 'refs/heads/main'  # Only on main branch
```

**Secrets:**
```yaml
${{ secrets.AWS_ACCESS_KEY_ID }}  # Encrypted in GitHub
```

---

## Part 5: Setting Up GitHub Secrets

### Required Secrets

Navigate to: **GitHub Repo → Settings → Secrets and variables → Actions**

**Add these secrets:**

1. **AWS_ACCESS_KEY_ID**
   - IAM user access key
   - Needs ECR push permissions

2. **AWS_SECRET_ACCESS_KEY**
   - IAM user secret key
   - Keep secure!

3. **GITHUB_TOKEN** (automatic)
   - Already provided by GitHub
   - For committing back to repo

### Create IAM User for GitHub Actions

```bash
# Create IAM user
aws iam create-user --user-name github-actions-ecr

# Create access key
aws iam create-access-key --user-name github-actions-ecr
# Save the output!

# Attach ECR policy
aws iam attach-user-policy \
  --user-name github-actions-ecr \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
```

**Or create custom policy (least privilege):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Part 6: Testing the Pipeline

### Trigger Pipeline

**Make a code change:**
```bash
# Edit UI code
echo "// Updated timestamp: $(date)" >> src/ui/src/main/java/com/retail/Application.java

# Commit and push
git add src/ui/
git commit -m "Update UI service - trigger CI/CD"
git push origin main
```

### Monitor Pipeline Execution

**In GitHub:**
1. Go to **Actions** tab
2. See running workflow
3. Click to view detailed logs

**Expected stages:**
```
✅ Build and Test (2-3 minutes)
✅ Build and Push Docker Image (3-5 minutes)
✅ Update Helm Values (30 seconds)
```

### Verify Image in ECR

```bash
# List images
aws ecr list-images \
  --repository-name retail-store/ui \
  --region us-west-2

# Describe image
aws ecr describe-images \
  --repository-name retail-store/ui \
  --region us-west-2
```

### Verify Helm Values Updated

```bash
# Pull latest changes
git pull origin main

# Check values.yaml
cat helm-charts/ui/values.yaml | grep tag
```

**Should show:**
```yaml
tag: "sha-abc1234"
```

### Verify ArgoCD Sync

```bash
# Check application status
argocd app get ui-service

# Watch sync progress
argocd app wait ui-service --sync --health --timeout 300
```

**In ArgoCD UI:**
1. Application shows "OutOfSync"
2. Auto-sync triggers (if enabled)
3. Shows "Synced" and "Healthy"

### Verify Deployment

```bash
# Check pods
kubectl get pods -n retail-store -l app=ui

# Verify image tag
kubectl get deployment ui -n retail-store -o jsonpath='{.spec.template.spec.containers[0].image}'

# Test application
kubectl port-forward svc/ui 8080:80 -n retail-store
# Open http://localhost:8080
```

---

## Part 7: Multi-Service Pipeline

### Create Workflows for All Services

**Catalog Service (Golang):**

`.github/workflows/catalog-service-ci.yml`:
```yaml
name: Catalog Service CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'src/catalog/**'

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: retail-store/catalog

jobs:
  build-push:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
          cache-dependency-path: src/catalog/go.sum
      
      - name: Run tests
        run: |
          cd src/catalog
          go test -v ./...
      
      - name: Build
        run: |
          cd src/catalog
          go build -v ./...
      
      # ... rest similar to UI service
```

**Checkout Service (Node.js):**

```yaml
name: Checkout Service CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'src/checkout/**'

jobs:
  build-push:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: src/checkout/package-lock.json
      
      - name: Install dependencies
        run: |
          cd src/checkout
          npm ci
      
      - name: Run tests
        run: |
          cd src/checkout
          npm test
      
      - name: Run linter
        run: |
          cd src/checkout
          npm run lint
      
      # ... rest similar to UI service
```

---

## Part 8: Environment-Specific Deployments

### Branch Strategy

```
main (production)
  ↓
  └─ dev (development)
       ↓
       └─ feature/* (features)
```

### Environment Workflows

**Development Environment:**

`.github/workflows/deploy-dev.yml`:
```yaml
name: Deploy to Dev

on:
  push:
    branches:
      - dev

env:
  ENVIRONMENT: dev

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Build and push with dev tag
      - name: Build and push
        # ... build steps ...
        with:
          tags: |
            ${{ secrets.ECR_REGISTRY }}/retail-store/ui:dev
            ${{ secrets.ECR_REGISTRY }}/retail-store/ui:dev-${{ github.sha }}
      
      # Update dev values
      - name: Update dev values
        run: |
          sed -i "s/tag: .*/tag: \"dev-${{ github.sha }}\"/" \
            helm-charts/ui/values-dev.yaml
          git add helm-charts/ui/values-dev.yaml
          git commit -m "Update dev image"
          git push
```

**Production Environment:**

Use main branch (from UI service workflow above)

### ArgoCD Applications Per Environment

**Dev Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ui-service-dev
  namespace: argocd
spec:
  project: retail-store
  source:
    repoURL: https://github.com/your-org/retail-store-app
    targetRevision: dev
    path: helm-charts/ui
    helm:
      valueFiles:
        - values-dev.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: retail-store-dev
```

**Prod Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ui-service-prod
  namespace: argocd
spec:
  project: retail-store
  source:
    repoURL: https://github.com/your-org/retail-store-app
    targetRevision: main
    path: helm-charts/ui
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: retail-store
  syncPolicy:
    automated:
      prune: false  # Manual prune in prod
      selfHeal: true
```

---

## Part 9: Advanced Pipeline Features

### Slack Notifications

Add to workflow:
```yaml
- name: Slack Notification
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment to production ${{ job.status }}'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

### Manual Approval for Production

```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://shop.example.com
    steps:
      # Deployment steps...
```

Then in GitHub:
1. Settings → Environments → production
2. Add "Required reviewers"
3. Deployment waits for approval

### Rollback Automation

```yaml
name: Rollback Production

on:
  workflow_dispatch:  # Manual trigger
    inputs:
      service:
        description: 'Service to rollback'
        required: true
        type: choice
        options:
          - ui
          - catalog
          - cart
      version:
        description: 'Version to rollback to'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Rollback image
        run: |
          sed -i "s/tag: .*/tag: \"${{ inputs.version }}\"/" \
            helm-charts/${{ inputs.service }}/values.yaml
          git add helm-charts/${{ inputs.service }}/values.yaml
          git commit -m "Rollback ${{ inputs.service }} to ${{ inputs.version }}"
          git push
```

---

## Part 10: Monitoring & Troubleshooting

### View Pipeline Logs

```bash
# Using GitHub CLI
gh run list
gh run view <run-id>
gh run view --log
```

### Common Issues

**Issue: ECR Push Failed**
```
Error: denied: User: arn:aws:iam::123456789012:user/github-actions is not authorized
```
**Solution:** Check IAM permissions

**Issue: Git Push Failed**
```
Error: refusing to allow a GitHub App to create or update workflow
```
**Solution:** Use Personal Access Token instead of GITHUB_TOKEN for workflow updates

**Issue: ArgoCD Not Syncing**
```bash
# Check ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server

# Force sync
argocd app sync ui-service --force
```

### Pipeline Metrics

**Track in GitHub:**
- Workflow run duration
- Success/failure rate
- Deployment frequency

**Add to workflow:**
```yaml
- name: Record metrics
  run: |
    echo "Build time: $SECONDS seconds"
    echo "deployment_time $SECONDS" >> metrics.txt
```

---

## 🎓 Key Takeaways

### What You Learned
1. ✅ Complete CI/CD pipeline with GitHub Actions
2. ✅ Automated Docker builds and ECR pushes
3. ✅ Helm value updates via automation
4. ✅ ArgoCD integration with Git
5. ✅ Zero-touch deployments
6. ✅ Environment-specific pipelines
7. ✅ Security scanning and approvals

### Pipeline Benefits
- **Speed**: Deployments in minutes
- **Reliability**: Automated testing
- **Traceability**: Full audit trail
- **Rollback**: Easy revert
- **Scale**: Deploy all services

---

## 🚦 Next Steps

**Next Tutorial:** Testing, Monitoring, and Production Best Practices
- Application testing
- Load testing
- Monitoring setup
- Cleanup procedures

---

## 📚 Quick Reference

```yaml
# GitHub Actions Syntax
on: push
jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello"

# Secrets
${{ secrets.SECRET_NAME }}

# Conditions
if: github.ref == 'refs/heads/main'
```

**Amazing work!** 🎉 You've built a complete CI/CD pipeline! One more tutorial to complete the project! 🚀
