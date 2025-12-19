# Tutorial 5: Microservices Architecture & Helm Charts

## 🎯 Learning Objectives
- Deep dive into microservices application architecture
- Understand Docker files for each service
- Learn Helm chart structure and templating
- Configure service-to-service communication
- Manage application secrets and config maps
- Deploy databases and caching layers

## 📋 Prerequisites
- Completed Tutorial 4 (EKS cluster running)
- kubectl configured and connected to cluster
- Helm 3.x installed
- Basic understanding of Docker and Kubernetes

---

## Part 1: Retail Store Application Architecture

### Complete System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                          User Browser                            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Nginx Ingress Controller                      │
│                  (External Load Balancer)                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│                        UI Service (Java)                         │
│                  Port 8080 - Frontend                            │
└──────────┬─────────┬──────────────┬──────────────┬──────────────┘
           │         │              │              │
    ┌──────┘         │              │              └──────┐
    │                │              │                     │
    ↓                ↓              ↓                     ↓
┌────────┐    ┌──────────┐   ┌──────────┐        ┌──────────┐
│Catalog │    │   Cart   │   │ Checkout │        │ Orders   │
│(Golang)│    │  (Java)  │   │ (Node.js)│        │  (Java)  │
│Port:80 │    │ Port:80  │   │ Port:80  │        │ Port:80  │
└───┬────┘    └─────┬────┘   └─────┬────┘        └────┬─────┘
    │               │              │                   │
    ↓               ↓              ↓                   ↓
┌────────┐    ┌──────────┐   ┌──────────┐      ┌──────────┐
│ MySQL  │    │ DynamoDB │   │  Redis   │      │PostgreSQL│
│        │    │          │   │  Cache   │      │          │
└────────┘    └──────────┘   └──────────┘      └──────────┘
```

### Service Responsibilities

**1. UI Service (Java/Spring Boot)**
- **Purpose**: User interface and frontend logic
- **Technology**: Java Spring Boot
- **Dependencies**: All backend services
- **Port**: 8080
- **Key Features**:
  - Product browsing
  - Shopping cart interface
  - Checkout flow
  - Order history display

**2. Catalog Service (Golang)**
- **Purpose**: Product catalog management
- **Technology**: Go (Golang)
- **Database**: MySQL
- **Port**: 80
- **Endpoints**:
  - `GET /products` - List all products
  - `GET /products/{id}` - Get product details
  - `POST /products` - Add new product

**3. Cart Service (Java)**
- **Purpose**: Shopping cart operations
- **Technology**: Java
- **Database**: DynamoDB (NoSQL)
- **Port**: 80
- **Key Features**:
  - Add items to cart
  - Remove items
  - Update quantities
  - Calculate totals

**4. Checkout Service (Node.js)**
- **Purpose**: Payment and order processing
- **Technology**: Node.js with Express
- **Cache**: Redis
- **Port**: 80
- **Key Features**:
  - Session management
  - Payment processing
  - Order creation

**5. Orders Service (Java)**
- **Purpose**: Order history and tracking
- **Technology**: Java
- **Database**: PostgreSQL
- **Port**: 80
- **Features**:
  - Order storage
  - Order retrieval
  - Order status updates

---

## Part 2: Understanding Docker Files

### UI Service Dockerfile

Located at: `src/ui/Dockerfile`

```dockerfile
# Multi-stage build for Java application

# Stage 1: Build stage
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source code and build
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Runtime stage
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Copy JAR from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Key Concepts:**

**Multi-stage Build:**
- **Stage 1 (builder)**: Compiles Java code with Maven
- **Stage 2 (runtime)**: Only includes compiled JAR
- **Benefit**: Smaller final image (JRE vs JDK)

**Why Alpine?**
- Minimal Linux distribution
- Image size: ~150MB vs ~500MB
- Security: Fewer packages = smaller attack surface

### Catalog Service Dockerfile

Located at: `src/catalog/Dockerfile`

```dockerfile
# Stage 1: Build Go binary
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build static binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o catalog .

# Stage 2: Runtime
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/catalog .

EXPOSE 80

# Run
CMD ["./catalog"]
```

**Go Binary Advantages:**
- Single executable file
- No runtime dependencies
- Extremely small image (~10-20MB)
- Fast startup time

### Checkout Service Dockerfile

Located at: `src/checkout/Dockerfile`

```dockerfile
# Node.js service
FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

EXPOSE 80

# Health check
HEALTHCHECK --interval=30s CMD node healthcheck.js

# Non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

CMD ["node", "server.js"]
```

**Security Best Practices:**
- Run as non-root user
- Use `npm ci` (clean install)
- Only production dependencies
- Health check endpoint

---

## Part 3: Helm Chart Structure

### What is a Helm Chart?

A Helm chart is a package containing:
- Kubernetes manifest templates
- Default configuration values
- Dependencies
- Documentation

### Typical Chart Structure

```
ui-service/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default configuration
├── values-dev.yaml         # Dev environment overrides
├── values-prod.yaml        # Prod environment overrides
├── templates/              # Kubernetes manifest templates
│   ├── deployment.yaml     # Deployment specification
│   ├── service.yaml        # Service definition
│   ├── ingress.yaml        # Ingress rules
│   ├── configmap.yaml      # Configuration data
│   ├── secret.yaml         # Sensitive data
│   ├── hpa.yaml           # Horizontal Pod Autoscaler
│   ├── serviceaccount.yaml # Service account
│   ├── _helpers.tpl        # Template helpers
│   └── NOTES.txt          # Post-install notes
└── charts/                 # Dependent charts
```

### Chart.yaml Example

```yaml
apiVersion: v2
name: ui-service
description: Retail store UI frontend service
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - retail
  - ecommerce
  - ui
maintainers:
  - name: Your Name
    email: your.email@example.com
dependencies:
  - name: redis
    version: "17.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
```

### values.yaml - Configuration Hub

```yaml
# Default values for ui-service

# Replica count
replicaCount: 2

# Image configuration
image:
  repository: public.ecr.aws/retail-store/ui
  pullPolicy: IfNotPresent
  tag: "1.0.0"

# Image pull secrets
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

# Service account
serviceAccount:
  create: true
  annotations: {}
  name: ""

# Pod annotations
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"

# Security context
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false

# Service configuration
service:
  type: ClusterIP
  port: 80
  targetPort: 8080

# Ingress configuration
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: shop.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: shop-tls
      hosts:
        - shop.example.com

# Resource limits
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

# Autoscaling
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

# Health checks
livenessProbe:
  httpGet:
    path: /actuator/health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: http
  initialDelaySeconds: 20
  periodSeconds: 5

# Environment variables
env:
  - name: SPRING_PROFILES_ACTIVE
    value: "production"
  - name: JAVA_OPTS
    value: "-Xmx256m -Xms128m"

# Service discovery endpoints
endpoints:
  catalog:
    url: http://catalog-service
  cart:
    url: http://cart-service
  checkout:
    url: http://checkout-service
  orders:
    url: http://orders-service

# Node selector
nodeSelector: {}

# Tolerations
tolerations: []

# Affinity rules
affinity: {}
```

---

## Part 4: Kubernetes Deployment Template

### deployment.yaml Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "ui-service.fullname" . }}
  labels:
    {{- include "ui-service.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "ui-service.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "ui-service.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "ui-service.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
      - name: {{ .Chart.Name }}
        securityContext:
          {{- toYaml .Values.securityContext | nindent 12 }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        livenessProbe:
          {{- toYaml .Values.livenessProbe | nindent 12 }}
        readinessProbe:
          {{- toYaml .Values.readinessProbe | nindent 12 }}
        env:
          {{- toYaml .Values.env | nindent 12 }}
          - name: CATALOG_URL
            value: {{ .Values.endpoints.catalog.url }}
          - name: CART_URL
            value: {{ .Values.endpoints.cart.url }}
          - name: CHECKOUT_URL
            value: {{ .Values.endpoints.checkout.url }}
          - name: ORDERS_URL
            value: {{ .Values.endpoints.orders.url }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

**Template Functions:**
- `{{ include "ui-service.fullname" . }}`: Generates resource name
- `{{- toYaml .Values.resources | nindent 12 }}`: Inserts YAML with indentation
- `{{ .Values.image.tag | default .Chart.AppVersion }}`: Uses tag or default

---

## Part 5: Service Discovery & Communication

### service.yaml Template

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "ui-service.fullname" . }}
  labels:
    {{- include "ui-service.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "ui-service.selectorLabels" . | nindent 4 }}
```

### How Services Communicate

**Within Kubernetes Cluster:**

```
UI Service calls: http://catalog-service/products
                              ↓
                    Kubernetes DNS resolves
                              ↓
                    Service: catalog-service
                              ↓
                    Load balances to pods
                              ↓
                    [Pod1] [Pod2] [Pod3]
```

**Service Discovery Example:**

```yaml
# In UI deployment
env:
  - name: CATALOG_URL
    value: "http://catalog-service"  # Service name as DNS
  - name: CART_URL
    value: "http://cart-service"
  - name: CHECKOUT_URL
    value: "http://checkout-service"
```

**Actual HTTP call from UI:**
```java
// Java code in UI service
String catalogUrl = System.getenv("CATALOG_URL");
RestTemplate restTemplate = new RestTemplate();
String response = restTemplate.getForObject(
    catalogUrl + "/products", 
    String.class
);
```

---

## Part 6: Database Configuration

### MySQL for Catalog Service

**Helm Chart:** Using Bitnami MySQL chart

```yaml
# values.yaml for catalog service
mysql:
  enabled: true
  auth:
    database: catalog
    username: catalog_user
    password: changeme123
  primary:
    persistence:
      enabled: true
      size: 10Gi
```

**Connection String:**
```yaml
env:
  - name: MYSQL_HOST
    value: "catalog-mysql"
  - name: MYSQL_PORT
    value: "3306"
  - name: MYSQL_DATABASE
    value: "catalog"
  - name: MYSQL_USER
    valueFrom:
      secretKeyRef:
        name: catalog-mysql
        key: username
  - name: MYSQL_PASSWORD
    valueFrom:
      secretKeyRef:
        name: catalog-mysql
        key: password
```

### PostgreSQL for Orders Service

```yaml
postgresql:
  enabled: true
  auth:
    database: orders
    username: orders_user
    password: securepass123
  primary:
    persistence:
      enabled: true
      size: 10Gi
    resources:
      requests:
        memory: 256Mi
        cpu: 250m
```

### Redis for Checkout Service

```yaml
redis:
  enabled: true
  architecture: standalone
  auth:
    enabled: true
    password: redispass123
  master:
    persistence:
      enabled: false  # In-memory cache
```

### DynamoDB for Cart Service

**Using AWS DynamoDB (managed service):**

```yaml
# IAM Role for Service Account (IRSA)
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/cart-dynamodb-role

env:
  - name: AWS_REGION
    value: us-west-2
  - name: DYNAMODB_TABLE_NAME
    value: retail-cart
```

**DynamoDB Table (create with Terraform):**
```hcl
resource "aws_dynamodb_table" "cart" {
  name           = "retail-cart"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "userId"
  range_key      = "productId"

  attribute {
    name = "userId"
    type = "S"
  }

  attribute {
    name = "productId"
    type = "S"
  }

  tags = {
    Name = "Retail Store Cart"
  }
}
```

---

## Part 7: ConfigMaps and Secrets

### ConfigMap Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "ui-service.fullname" . }}-config
data:
  application.yaml: |
    server:
      port: 8080
    
    logging:
      level:
        root: INFO
        com.retail: DEBUG
    
    management:
      endpoints:
        web:
          exposure:
            include: health,info,metrics
      
      health:
        readiness-state:
          enabled: true
        liveness-state:
          enabled: true
    
    services:
      catalog:
        url: {{ .Values.endpoints.catalog.url }}
      cart:
        url: {{ .Values.endpoints.cart.url }}
      checkout:
        url: {{ .Values.endpoints.checkout.url }}
      orders:
        url: {{ .Values.endpoints.orders.url }}
```

### Secret Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "ui-service.fullname" . }}-secret
type: Opaque
stringData:
  db-password: {{ .Values.database.password | b64enc }}
  api-key: {{ .Values.apiKeys.payment | b64enc }}
```

**Using Secrets in Pods:**

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: ui-service-secret
        key: db-password
```

---

## Part 8: Deploying with Helm

### Install Catalog Service

```bash
# Add Bitnami repo for MySQL
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Create namespace
kubectl create namespace retail-store

# Install catalog service
helm install catalog ./helm-charts/catalog \
  --namespace retail-store \
  --values ./helm-charts/catalog/values-dev.yaml \
  --set image.tag=v1.0.0
```

**Monitor deployment:**
```bash
# Watch pods
kubectl get pods -n retail-store --watch

# Check helm release
helm list -n retail-store

# View release details
helm get all catalog -n retail-store
```

### Install All Services

```bash
# Install in order (databases first, then services)

# 1. Catalog (with MySQL)
helm install catalog ./helm-charts/catalog -n retail-store

# 2. Cart (needs DynamoDB access)
helm install cart ./helm-charts/cart -n retail-store

# 3. Checkout (with Redis)
helm install checkout ./helm-charts/checkout -n retail-store

# 4. Orders (with PostgreSQL)
helm install orders ./helm-charts/orders -n retail-store

# 5. UI (frontend)
helm install ui ./helm-charts/ui -n retail-store
```

### Verify All Services

```bash
# Check all pods
kubectl get pods -n retail-store

# Expected output:
# NAME                         READY   STATUS    RESTARTS   AGE
# catalog-5d8f6b7c9d-abc12     1/1     Running   0          2m
# catalog-mysql-0              1/1     Running   0          2m
# cart-7b9c8d6e5f-def34        1/1     Running   0          2m
# checkout-8c0d9e7f6g-ghi56    1/1     Running   0          2m
# checkout-redis-master-0      1/1     Running   0          2m
# orders-9d1e0f8g7h-jkl78      1/1     Running   0          2m
# orders-postgresql-0          1/1     Running   0          2m
# ui-0e2f1g9h8i-mno90          1/1     Running   0          2m

# Check services
kubectl get svc -n retail-store

# Test service connectivity
kubectl run test-pod --image=busybox -n retail-store --rm -it -- /bin/sh
# Inside pod:
wget -O- http://catalog-service/products
```

---

## Part 9: Customizing with values.yaml

### Environment-Specific Values

**values-dev.yaml:**
```yaml
replicaCount: 1

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi

autoscaling:
  enabled: false

ingress:
  enabled: false

mysql:
  primary:
    persistence:
      enabled: false  # Faster for dev
```

**values-prod.yaml:**
```yaml
replicaCount: 3

resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20

ingress:
  enabled: true
  hosts:
    - host: shop.production.com

mysql:
  primary:
    persistence:
      enabled: true
      size: 100Gi
    resources:
      requests:
        cpu: 1000m
        memory: 2Gi
```

### Deploy with Custom Values

```bash
# Development
helm install catalog ./helm-charts/catalog \
  -n retail-store \
  -f ./helm-charts/catalog/values-dev.yaml

# Production
helm install catalog ./helm-charts/catalog \
  -n retail-store \
  -f ./helm-charts/catalog/values-prod.yaml

# Override specific values
helm install catalog ./helm-charts/catalog \
  -n retail-store \
  --set replicaCount=5 \
  --set image.tag=v2.0.0
```

---

## Part 10: Troubleshooting Deployments

### Common Issues & Solutions

**Pod Not Starting:**
```bash
# Describe pod
kubectl describe pod <pod-name> -n retail-store

# Common issues:
# - ImagePullBackOff: Wrong image name/tag
# - CrashLoopBackOff: App crashes on start
# - Pending: No nodes available
```

**Database Connection Issues:**
```bash
# Check if database pod is running
kubectl get pods -l app=mysql -n retail-store

# Test database connection from app pod
kubectl exec -it <app-pod> -n retail-store -- /bin/sh
# Inside pod:
nc -zv catalog-mysql 3306
```

**Service Discovery Problems:**
```bash
# Check service exists
kubectl get svc catalog-service -n retail-store

# Test DNS resolution
kubectl run test-dns --image=busybox -n retail-store --rm -it -- nslookup catalog-service
```

**View Logs:**
```bash
# Pod logs
kubectl logs <pod-name> -n retail-store

# Follow logs
kubectl logs -f <pod-name> -n retail-store

# Previous pod logs (if crashed)
kubectl logs <pod-name> -n retail-store --previous
```

---

## 🎓 Key Takeaways

### What You Learned
1. ✅ Microservices architecture patterns
2. ✅ Docker multi-stage builds
3. ✅ Helm chart structure and templating
4. ✅ Service discovery in Kubernetes
5. ✅ Database deployment strategies
6. ✅ ConfigMaps and Secrets management
7. ✅ Helm deployment workflows
8. ✅ Troubleshooting techniques

---

## 🚦 Next Steps

**Next Tutorial:** ArgoCD Setup & GitOps Deployment
- Install ArgoCD on EKS
- Configure Git repositories
- Create ArgoCD applications
- Implement automated sync

---

## 📚 Quick Reference

```bash
# Helm Commands
helm install <name> <chart> -n <namespace>
helm upgrade <name> <chart> -n <namespace>
helm uninstall <name> -n <namespace>
helm list -n <namespace>
helm get values <name> -n <namespace>

# Debugging
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl exec -it <pod> -n <namespace> -- /bin/sh
kubectl port-forward svc/<service> 8080:80 -n <namespace>
```

**Great job!** 🎉 You understand microservices deployment with Helm. Next: GitOps automation! 🚀
