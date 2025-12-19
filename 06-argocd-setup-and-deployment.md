# Tutorial 6: ArgoCD Setup & GitOps Deployment

## 🎯 Learning Objectives
- Understand GitOps principles and benefits
- Install and configure ArgoCD on EKS
- Create ArgoCD projects and applications
- Implement automatic synchronization from Git
- Configure application health monitoring
- Use ArgoCD UI for deployment management

## 📋 Prerequisites
- Completed Tutorial 5 (Helm charts ready)
- EKS cluster running and accessible
- kubectl configured
- Helm charts committed to Git repository

---

## Part 1: Understanding GitOps

### What is GitOps?

**GitOps** is a deployment methodology where:
- Git is the single source of truth
- Desired state is declared in Git
- Automated processes sync Git to cluster
- Changes happen through Git commits/PRs

### Traditional Deployment vs GitOps

**Traditional (Manual):**
```
Developer → kubectl apply → Cluster
Developer → helm install → Cluster
Developer → terraform apply → Infrastructure
```
**Problems:**
- ❌ No audit trail
- ❌ Hard to rollback
- ❌ Difficult collaboration
- ❌ Manual processes
- ❌ Drift between environments

**GitOps (Automated):**
```
Developer → Git Commit → Pull Request → Merge
                              ↓
                         ArgoCD watches Git
                              ↓
                    Automatically syncs to Cluster
```
**Benefits:**
- ✅ Complete audit trail (Git history)
- ✅ Easy rollback (Git revert)
- ✅ Collaboration through PRs
- ✅ Automated deployment
- ✅ Self-healing (auto-sync from Git)

### GitOps Workflow

```
┌──────────────┐
│  Developer   │
└──────┬───────┘
       │ git push
       ↓
┌──────────────┐
│ Git Repo     │ ← Single Source of Truth
│ (GitHub)     │
└──────┬───────┘
       │ Monitors
       ↓
┌──────────────┐
│   ArgoCD     │ ← Continuous Deployment
│              │
└──────┬───────┘
       │ Syncs
       ↓
┌──────────────┐
│ EKS Cluster  │ ← Production Environment
└──────────────┘
```

---

## Part 2: What is ArgoCD?

### ArgoCD Overview

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes.

**Key Features:**
1. **Automated Deployment**: Watches Git and syncs to cluster
2. **Application Definitions**: Declarative app configurations
3. **Health Status**: Monitors application health
4. **Sync Policies**: Manual or automatic sync
5. **Rollback**: Easy revert to previous versions
6. **Multi-cluster Support**: Manage multiple clusters
7. **SSO Integration**: RBAC and authentication
8. **Web UI**: Visual deployment management

### ArgoCD Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    ArgoCD Components                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────┐          ┌──────────────────┐   │
│  │  API Server      │◄─────────│  Web UI          │   │
│  │  (REST/gRPC)     │          │  (Dashboard)     │   │
│  └────────┬─────────┘          └──────────────────┘   │
│           │                                             │
│  ┌────────▼─────────┐          ┌──────────────────┐   │
│  │ Repo Server      │          │  Redis (Cache)   │   │
│  │ (Git Poller)     │◄─────────│                  │   │
│  └────────┬─────────┘          └──────────────────┘   │
│           │                                             │
│  ┌────────▼─────────┐                                  │
│  │ Application      │                                  │
│  │ Controller       │                                  │
│  │ (Sync Engine)    │                                  │
│  └────────┬─────────┘                                  │
└───────────┼─────────────────────────────────────────────┘
            │
            ↓
   ┌────────────────┐
   │  Kubernetes    │
   │  API Server    │
   └────────────────┘
```

---

## Part 3: Installing ArgoCD on EKS

### Method 1: Using kubectl (Quick Start)

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**What gets installed:**
- ArgoCD API Server
- Repository Server
- Application Controller
- Redis
- Dex (for SSO)
- Server UI

**Wait for pods to be ready:**
```bash
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Check status
kubectl get pods -n argocd
```

**Expected Output:**
```
NAME                                  READY   STATUS    RESTARTS   AGE
argocd-application-controller-0       1/1     Running   0          2m
argocd-dex-server-5dd657bd9-xxxxx     1/1     Running   0          2m
argocd-redis-74cb89f466-xxxxx         1/1     Running   0          2m
argocd-repo-server-6b76b9d5f7-xxxxx   1/1     Running   0          2m
argocd-server-7f5b8b9d9c-xxxxx        1/1     Running   0          2m
```

### Method 2: Using Helm (Recommended for Production)

```bash
# Add Argo Helm repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Create values file for customization
cat << 'EOF' > argocd-values.yaml
server:
  replicas: 2
  service:
    type: LoadBalancer
  
  extraArgs:
    - --insecure  # For learning (use TLS in production)
  
  config:
    repositories: |
      - type: git
        url: https://github.com/your-org/retail-store-app
      - type: helm
        name: stable
        url: https://charts.helm.sh/stable

controller:
  replicas: 1

repoServer:
  replicas: 2

redis:
  enabled: true

configs:
  params:
    server.insecure: true  # Disable TLS for learning
EOF

# Install with Helm
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --values argocd-values.yaml
```

### Method 3: Using Terraform (Infrastructure as Code)

Add to your Terraform configuration:

```hcl
resource "kubernetes_namespace" "argocd" {
  metadata {
    name = "argocd"
  }
}

resource "helm_release" "argocd" {
  name       = "argocd"
  repository = "https://argoproj.github.io/argo-helm"
  chart      = "argo-cd"
  namespace  = kubernetes_namespace.argocd.metadata[0].name
  version    = "5.51.0"

  values = [
    file("${path.module}/argocd-values.yaml")
  ]

  depends_on = [module.eks]
}
```

Then apply:
```bash
terraform apply -target=helm_release.argocd
```

---

## Part 4: Accessing ArgoCD UI

### Get Initial Admin Password

```bash
# The initial password is auto-generated
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

**Save this password!** Username is: `admin`

### Access ArgoCD Server

**Option 1: Port Forward (Quick)**
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open: http://localhost:8080

**Option 2: Load Balancer (Production)**
```bash
# Patch service to LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get external URL
kubectl get svc argocd-server -n argocd
```

**Option 3: Ingress (Best Practice)**

Create `argocd-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-server-tls
```

Apply:
```bash
kubectl apply -f argocd-ingress.yaml
```

### Install ArgoCD CLI (Optional but Useful)

**Linux:**
```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
```

**macOS:**
```bash
brew install argocd
```

**Windows:**
```powershell
choco install argocd-cli
```

**Login via CLI:**
```bash
# Port forward first
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login
argocd login localhost:8080 \
  --username admin \
  --password <your-password> \
  --insecure

# Change password
argocd account update-password
```

---

## Part 5: Creating ArgoCD Project

### What is an ArgoCD Project?

**Projects** provide logical grouping and RBAC for applications:
- Group related applications
- Define source repositories
- Restrict destination clusters/namespaces
- Set resource whitelist/blacklist

### Create Project via UI

1. Login to ArgoCD UI
2. Click "Settings" → "Projects"
3. Click "+ New Project"
4. Fill in details:
   - **Name**: `retail-store`
   - **Description**: `Retail store microservices`
   - **Sources**: Add your Git repository
   - **Destinations**: 
     - Cluster: `https://kubernetes.default.svc`
     - Namespace: `retail-store`

### Create Project via YAML

Create `argocd-project.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: retail-store
  namespace: argocd
spec:
  description: Retail Store Application Project
  
  # Source repositories
  sourceRepos:
    - 'https://github.com/your-org/retail-store-app'
    - 'https://charts.bitnami.com/bitnami'
  
  # Destination clusters and namespaces
  destinations:
    - namespace: 'retail-store'
      server: 'https://kubernetes.default.svc'
    - namespace: 'retail-store-*'  # Wildcard for multiple envs
      server: 'https://kubernetes.default.svc'
  
  # Cluster resource whitelist
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRole
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRoleBinding
  
  # Namespace resource whitelist (empty = all allowed)
  namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
  
  # Namespace resource blacklist
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
    - group: ''
      kind: LimitRange
  
  # Orphaned resources (resources not in Git)
  orphanedResources:
    warn: true
  
  # Roles for RBAC
  roles:
    - name: read-only
      description: Read-only access to applications
      policies:
        - p, proj:retail-store:read-only, applications, get, retail-store/*, allow
      groups:
        - developers
    
    - name: admin
      description: Full access to applications
      policies:
        - p, proj:retail-store:admin, applications, *, retail-store/*, allow
      groups:
        - ops-team
```

Apply:
```bash
kubectl apply -f argocd-project.yaml
```

### Create Project via CLI

```bash
argocd proj create retail-store \
  --description "Retail Store Application Project" \
  --src https://github.com/your-org/retail-store-app \
  --dest https://kubernetes.default.svc,retail-store \
  --orphaned-resources-warn
```

---

## Part 6: Creating ArgoCD Applications

### Application Structure

Each microservice gets its own ArgoCD Application:
- `ui-application`
- `catalog-application`
- `cart-application`
- `checkout-application`
- `orders-application`

### Create Application via UI

1. Click "+ New App" in UI
2. Fill in:
   - **Application Name**: `ui-service`
   - **Project**: `retail-store`
   - **Sync Policy**: `Automatic`
   - **Repository URL**: `https://github.com/your-org/retail-store-app`
   - **Path**: `helm-charts/ui`
   - **Cluster URL**: `https://kubernetes.default.svc`
   - **Namespace**: `retail-store`
3. Click "Create"

### Create Application via YAML

Create `ui-application.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ui-service
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io  # Cascade delete
spec:
  project: retail-store
  
  # Source (Git repository)
  source:
    repoURL: https://github.com/your-org/retail-store-app
    targetRevision: main  # Branch or tag
    path: helm-charts/ui
    
    # Helm-specific options
    helm:
      releaseName: ui
      
      # Override values
      values: |
        replicaCount: 2
        image:
          tag: v1.0.0
        resources:
          requests:
            cpu: 250m
            memory: 256Mi
      
      # Or use values file from repo
      valueFiles:
        - values-prod.yaml
  
  # Destination (Kubernetes cluster)
  destination:
    server: https://kubernetes.default.svc
    namespace: retail-store
  
  # Sync policy
  syncPolicy:
    automated:
      prune: true      # Delete resources not in Git
      selfHeal: true   # Auto-sync when cluster state drifts
      allowEmpty: false
    
    syncOptions:
      - CreateNamespace=true  # Auto-create namespace
      - PruneLast=true       # Delete resources last
    
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  # Health check
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # Ignore replica count (for HPA)
```

Apply all applications:
```bash
kubectl apply -f ui-application.yaml
kubectl apply -f catalog-application.yaml
kubectl apply -f cart-application.yaml
kubectl apply -f checkout-application.yaml
kubectl apply -f orders-application.yaml
```

### Create All Applications with Kustomize

Create `argocd/applications/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ui-application.yaml
  - catalog-application.yaml
  - cart-application.yaml
  - checkout-application.yaml
  - orders-application.yaml
```

Apply:
```bash
kubectl apply -k argocd/applications/
```

---

## Part 7: App of Apps Pattern

### What is App of Apps?

A parent application that creates child applications:
```
parent-app (ArgoCD Application)
  ├── ui-application
  ├── catalog-application
  ├── cart-application
  ├── checkout-application
  └── orders-application
```

### Create Parent Application

Create `argocd/app-of-apps.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: retail-store-apps
  namespace: argocd
spec:
  project: retail-store
  
  source:
    repoURL: https://github.com/your-org/retail-store-app
    targetRevision: main
    path: argocd/applications  # Directory with all app definitions
  
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd  # Applications go in argocd namespace
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Single command to deploy everything:**
```bash
kubectl apply -f argocd/app-of-apps.yaml
```

ArgoCD will:
1. Read app-of-apps.yaml
2. Create all child applications
3. Deploy all services automatically

---

## Part 8: Monitoring & Managing Applications

### ArgoCD UI Features

**Application Tile:**
- 🟢 Green: Healthy and synced
- 🟡 Yellow: Out of sync
- 🔴 Red: Degraded/Failed

**Resource View:**
- Visualize all Kubernetes resources
- Click to see YAML and events
- See relationships between resources

**Application Health:**
```
✅ Healthy: All resources running
❌ Degraded: Some resources failing
🕐 Progressing: Deployment in progress
⚠️  Missing: Resources not created
❓ Unknown: Health status unknown
```

### Sync Status

**Synced**: Git matches cluster
**OutOfSync**: Git differs from cluster

### Manual Sync

**Via UI:**
1. Click application
2. Click "Sync" button
3. Select resources to sync
4. Click "Synchronize"

**Via CLI:**
```bash
# Sync specific app
argocd app sync ui-service

# Sync all apps in project
argocd app sync -l project=retail-store

# Dry run (preview)
argocd app sync ui-service --dry-run

# Sync specific resource
argocd app sync ui-service --resource=:Deployment:ui
```

### Rollback

**Via UI:**
1. Click application
2. Go to "History and Rollback"
3. Select previous revision
4. Click "Rollback"

**Via CLI:**
```bash
# List sync history
argocd app history ui-service

# Rollback to previous
argocd app rollback ui-service

# Rollback to specific revision
argocd app rollback ui-service 5
```

### Application Logs

```bash
# View logs
argocd app logs ui-service

# Follow logs
argocd app logs ui-service --follow

# Specific container
argocd app logs ui-service --container=ui
```

---

## Part 9: GitOps Workflow in Action

### Making Changes

**Workflow:**
```
1. Developer changes code
   ↓
2. Commit to feature branch
   ↓
3. Create Pull Request
   ↓
4. Code review
   ↓
5. Merge to main branch
   ↓
6. ArgoCD detects change (within 3 minutes)
   ↓
7. ArgoCD syncs to cluster
   ↓
8. Application updated automatically
```

### Example: Update Image Tag

**Edit values.yaml:**
```yaml
image:
  repository: public.ecr.aws/retail-store/ui
  tag: v1.0.1  # Changed from v1.0.0
```

**Commit and push:**
```bash
git add helm-charts/ui/values.yaml
git commit -m "Update UI to v1.0.1"
git push origin main
```

**ArgoCD automatically:**
1. Detects change in Git
2. Shows "OutOfSync" status
3. Syncs if auto-sync enabled
4. Performs rolling update
5. Shows "Synced" and "Healthy"

**Monitor in real-time:**
```bash
# Watch application status
watch argocd app get ui-service

# Watch pods rolling update
kubectl get pods -n retail-store -w
```

---

## Part 10: Advanced Configurations

### Sync Waves

Control deployment order with annotations:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "2"  # Deploy in wave 2
```

**Example order:**
- Wave 0: Namespaces, ConfigMaps, Secrets
- Wave 1: Databases (MySQL, PostgreSQL, Redis)
- Wave 2: Backend services (catalog, cart, orders)
- Wave 3: Frontend (UI)
- Wave 4: Ingress

### Resource Hooks

Execute tasks before/after sync:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: db-migrate
        image: flyway:latest
        command: ["flyway", "migrate"]
      restartPolicy: Never
```

**Hook types:**
- `PreSync`: Before sync
- `Sync`: During sync
- `PostSync`: After sync
- `SyncFail`: If sync fails
- `Skip`: Never sync

### Notifications

Configure Slack/email notifications:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} is now running new version.
  
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-deployed]
```

---

## 🎓 Key Takeaways

### What You Learned
1. ✅ GitOps principles and benefits
2. ✅ ArgoCD architecture and components
3. ✅ Installing ArgoCD on EKS
4. ✅ Creating projects and applications
5. ✅ Automated sync from Git
6. ✅ Monitoring and rollback
7. ✅ App of Apps pattern
8. ✅ Advanced sync configurations

### GitOps Benefits Achieved
- **Audit Trail**: Every change in Git
- **Easy Rollback**: Git revert
- **Self-Healing**: Auto-sync from Git
- **Declarative**: All config in code
- **Automated**: No manual kubectl

---

## 🚦 Next Steps

**Next Tutorial:** Complete CI/CD Pipeline with GitHub Actions
- Build Docker images
- Push to ECR
- Update Helm values
- Trigger ArgoCD sync

---

## 📚 Quick Reference

```bash
# ArgoCD CLI
argocd app list
argocd app get <app-name>
argocd app sync <app-name>
argocd app rollback <app-name>
argocd app logs <app-name> --follow

# Application Status
argocd app wait <app-name>
argocd app diff <app-name>
argocd app manifests <app-name>

# Projects
argocd proj list
argocd proj create <name>
```

**Fantastic progress!** 🎉 You've implemented GitOps! Next: Full CI/CD automation! 🚀
