# Kubernetes to Production Tutorial Series

## 📚 Complete Tutorial Guide

A comprehensive, beginner-friendly tutorial series covering production-ready Kubernetes deployment using AWS EKS Auto Mode, ArgoCD, Terraform, and GitOps practices.

---

## 🎯 What You'll Build

A **complete production-grade e-commerce application** featuring:

- **Multi-tier microservices architecture** (5 services)
- **AWS EKS cluster** with Auto Mode
- **GitOps deployment** with ArgoCD
- **Full CI/CD pipeline** with GitHub Actions
- **Infrastructure as Code** with Terraform
- **Container registry** with AWS ECR

---

## 📖 Tutorial Series

### Tutorial 1: [Introduction & Project Overview](01-introduction-and-project-overview.md)
**Duration:** 15-20 minutes reading

**What you'll learn:**
- Complete project architecture
- Microservices vs monolithic design
- EKS Auto Mode benefits
- Infrastructure stack overview
- Learning outcomes

**Prerequisites:** None

---

### Tutorial 2: [AWS CLI Setup & EKS Auto Mode](02-aws-cli-and-eks-setup.md)
**Duration:** 30-45 minutes

**What you'll learn:**
- AWS CLI installation and configuration
- IAM user creation with proper permissions
- Understanding EKS vs EKS Auto Mode
- Creating first EKS cluster manually
- Verifying cluster access

**Prerequisites:**
- AWS account (free tier eligible)
- Terminal/command line access

---

### Tutorial 3: [Prerequisites & Tool Installation](03-prerequisites-and-tool-installation.md)
**Duration:** 45-60 minutes

**What you'll learn:**
- Installing Terraform, kubectl, Helm
- Configuring development environment
- Setting up IDE with helpful extensions
- Repository structure
- Tool verification

**Prerequisites:**
- Completed Tutorial 2
- Basic command line knowledge

---

### Tutorial 4: [Terraform Infrastructure Deployment](04-terraform-vpc-eks-deployment.md)
**Duration:** 60-90 minutes

**What you'll learn:**
- Terraform project structure
- Creating VPC with public/private subnets
- Deploying EKS cluster with Terraform
- Understanding IAM roles and policies
- Managing Terraform state
- Cost considerations

**Prerequisites:**
- Completed Tutorial 3
- Terraform installed and working

---

### Tutorial 5: [Microservices Architecture & Helm Charts](05-microservices-and-helm-architecture.md)
**Duration:** 90-120 minutes

**What you'll learn:**
- Deep dive into microservices architecture
- Understanding Docker multi-stage builds
- Helm chart structure and templating
- Service-to-service communication
- Database deployment strategies
- ConfigMaps and Secrets management

**Prerequisites:**
- Completed Tutorial 4
- EKS cluster running
- Basic Docker knowledge

---

### Tutorial 6: [ArgoCD Setup & GitOps Deployment](06-argocd-setup-and-deployment.md)
**Duration:** 60-90 minutes

**What you'll learn:**
- GitOps principles and benefits
- Installing and configuring ArgoCD
- Creating ArgoCD projects and applications
- Automated synchronization from Git
- App of Apps pattern
- Monitoring and rollback procedures

**Prerequisites:**
- Completed Tutorial 5
- Helm charts ready in Git repository

---

### Tutorial 7: [Complete CI/CD Pipeline with GitOps](07-cicd-pipeline-with-gitops.md)
**Duration:** 90-120 minutes

**What you'll learn:**
- Building GitHub Actions workflows
- Automated Docker image building
- Pushing images to AWS ECR
- Automated Helm value updates
- Zero-touch deployments
- Environment-specific pipelines

**Prerequisites:**
- Completed Tutorial 6
- GitHub account
- ArgoCD configured

---

### Tutorial 8: [Testing, Monitoring & Production Cleanup](08-testing-monitoring-and-cleanup.md)
**Duration:** 60-90 minutes

**What you'll learn:**
- End-to-end application testing
- Verifying shopping cart and checkout flow
- ArgoCD dashboard monitoring
- Troubleshooting common issues
- Production best practices checklist
- Complete cleanup procedures
- Resume building tips

**Prerequisites:**
- Completed Tutorial 7
- All services deployed

---
## Github repo
https://github.com/islamMaruf/retail-store-sample-app
## 🛠️ Technology Stack

| Category | Technology |
|----------|------------|
| **Container Orchestration** | Kubernetes (AWS EKS) |
| **Cloud Platform** | AWS (EKS, ECR, VPC, IAM) |
| **Infrastructure as Code** | Terraform |
| **GitOps CD** | ArgoCD |
| **CI/CD** | GitHub Actions |
| **Package Management** | Helm |
| **Container Runtime** | Docker |
| **Languages** | Java, Golang, Node.js |
| **Databases** | MySQL, PostgreSQL, Redis, DynamoDB |

---

## 📋 Complete Prerequisites

### Required Knowledge
- ✅ Basic Linux command line
- ✅ Git version control basics
- ✅ Docker fundamentals (containers, images)
- ✅ Basic Kubernetes concepts (pods, services, deployments)
- ✅ Terraform basics (helpful but not required)

### Required Accounts
- ✅ AWS Account (free tier eligible)
- ✅ GitHub Account
- ✅ Terminal/Command Line access

### Required Tools (will install in Tutorial 3)
- AWS CLI 2.x
- Terraform 1.5+
- kubectl 1.31+
- Helm 3.x
- Git 2.x
- Docker (optional)

---

## ⏱️ Time Commitment

| Tutorial | Reading | Hands-on | Total |
|----------|---------|----------|-------|
| Tutorial 1 | 20 min | 0 min | 20 min |
| Tutorial 2 | 20 min | 30 min | 50 min |
| Tutorial 3 | 30 min | 45 min | 75 min |
| Tutorial 4 | 30 min | 60 min | 90 min |
| Tutorial 5 | 45 min | 90 min | 135 min |
| Tutorial 6 | 30 min | 60 min | 90 min |
| Tutorial 7 | 45 min | 90 min | 135 min |
| Tutorial 8 | 30 min | 60 min | 90 min |
| **Total** | **4 hours** | **7 hours** | **~11 hours** |

---

## 💰 Cost Estimate

**AWS Resources:**
- EKS Control Plane: $0.10/hour (~$72/month)
- EKS Auto Mode Nodes: ~$20-50/month (scales automatically)
- NAT Gateways: ~$32/month each (2 total = $64)
- Load Balancers: ~$20/month
- **Estimated Total: $150-200/month**

**Cost Savings Tips:**
- ✅ Use AWS Free Tier credits ($100-200 for new accounts)
- ✅ Destroy resources immediately after learning
- ✅ Use single NAT gateway for learning
- ✅ Set up billing alerts
- ✅ Complete tutorial in focused sessions

**Cleanup is covered in Tutorial 8 to minimize costs!**

---

## 🎯 Learning Outcomes

By completing this tutorial series, you will:

### Technical Skills
1. ✅ Deploy production-ready EKS clusters with Auto Mode
2. ✅ Create infrastructure using Terraform
3. ✅ Build and containerize microservices
4. ✅ Design Helm charts for applications
5. ✅ Implement GitOps with ArgoCD
6. ✅ Build CI/CD pipelines with GitHub Actions
7. ✅ Manage AWS cloud resources
8. ✅ Troubleshoot Kubernetes deployments

### DevOps Practices
1. ✅ Infrastructure as Code methodology
2. ✅ GitOps deployment strategies
3. ✅ Container orchestration patterns
4. ✅ Microservices architecture
5. ✅ Automated testing and deployment
6. ✅ Monitoring and observability
7. ✅ Security best practices

### Career Benefits
- Portfolio-ready project for resume
- Real-world production experience
- Interview-ready technical knowledge
- Certification preparation (CKA/CKAD)
- High-demand skills (DevOps/Cloud)

---

## 📁 Project Structure

```
retail-store-kubernetes/
├── tutorials/                          # Tutorial documentation
│   ├── 01-introduction-and-project-overview.md
│   ├── 02-aws-cli-and-eks-setup.md
│   ├── 03-prerequisites-and-tool-installation.md
│   ├── 04-terraform-vpc-eks-deployment.md
│   ├── 05-microservices-and-helm-architecture.md
│   ├── 06-argocd-setup-and-deployment.md
│   ├── 07-cicd-pipeline-with-gitops.md
│   └── 08-testing-monitoring-and-cleanup.md
├── terraform/                          # Infrastructure as Code
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── modules/
│       ├── vpc/
│       └── eks/
├── helm-charts/                        # Kubernetes application charts
│   ├── ui/
│   ├── catalog/
│   ├── cart/
│   ├── checkout/
│   └── orders/
├── src/                                # Application source code
│   ├── ui/                            # Java Spring Boot
│   ├── catalog/                       # Golang
│   ├── cart/                          # Java
│   ├── checkout/                      # Node.js
│   └── orders/                        # Java
├── .github/workflows/                  # CI/CD pipelines
│   ├── ui-service-ci.yml
│   ├── catalog-service-ci.yml
│   ├── cart-service-ci.yml
│   ├── checkout-service-ci.yml
│   └── orders-service-ci.yml
├── argocd/                            # GitOps configurations
│   ├── projects/
│   └── applications/
└── README.md                          # This file
```

---

## 🚀 Getting Started

### Quick Start Guide

1. **Start with Tutorial 1** to understand the project
2. **Follow tutorials in order** (each builds on previous)
3. **Complete hands-on exercises** for best learning
4. **Don't skip cleanup** in Tutorial 8 (avoid AWS charges)
5. **Use GitHub for version control** throughout

### Recommended Approach

**Week 1: Foundations**
- Complete Tutorials 1-3
- Set up all tools and accounts
- Get comfortable with AWS CLI

**Week 2: Infrastructure**
- Complete Tutorial 4
- Build infrastructure with Terraform
- Understand EKS cluster management

**Week 3: Applications**
- Complete Tutorials 5-6
- Deploy microservices with Helm
- Set up ArgoCD and GitOps

**Week 4: Automation**
- Complete Tutorials 7-8
- Build CI/CD pipelines
- Test, monitor, and cleanup

---

## 🎓 Certification Preparation

This tutorial helps prepare for:

**Certified Kubernetes Administrator (CKA)**
- Cluster architecture
- Workload management
- Services and networking
- Storage
- Troubleshooting

**Certified Kubernetes Application Developer (CKAD)**
- Application design and build
- Application deployment
- Application observability and maintenance
- Application environment, configuration

**AWS Certified Solutions Architect**
- AWS EKS
- VPC networking
- IAM security
- Infrastructure as Code

---

## 📚 Additional Resources

### Documentation
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/)
- [Helm Documentation](https://helm.sh/docs/)

### Community
- [Kubernetes Slack](https://kubernetes.slack.com/)
- [CNCF Community](https://www.cncf.io/community/)
- [r/kubernetes](https://reddit.com/r/kubernetes)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/kubernetes)

### Learning Platforms
- [KillerCoda Interactive Labs](https://killercoda.com/)
- [Kubernetes By Example](https://kubernetesbyexample.com/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

---

## 🤝 Contributing

Found an issue or have suggestions?
- Open an issue on GitHub
- Submit a pull request
- Share feedback

---

## 📞 Support

Need help?
1. Review the specific tutorial's troubleshooting section
2. Check Tutorial 8 for common issues
3. Search GitHub issues
4. Ask in Kubernetes Slack
5. Post on Stack Overflow with tags: `kubernetes`, `aws-eks`, `argocd`

---

## ⚠️ Important Notes

### Cost Management
- **Always cleanup resources** after completing tutorials
- Set up AWS billing alerts
- Monitor costs in AWS Cost Explorer
- Tutorial 8 covers complete cleanup

### Security
- **Never commit secrets** to Git repositories
- Use AWS Secrets Manager for production
- Follow principle of least privilege
- Keep tools and dependencies updated

### Production Considerations
- This tutorial is for learning purposes
- Additional hardening needed for production
- Implement proper monitoring/alerting
- Add backup/disaster recovery
- Review AWS Well-Architected Framework

---

## 🎉 Success Stories

After completing this tutorial series, you'll be able to:

✅ Confidently discuss Kubernetes in interviews
✅ Deploy production applications to the cloud
✅ Implement modern DevOps practices
✅ Contribute to Kubernetes projects
✅ Mentor others in containerization
✅ Pursue cloud-native career opportunities

---

## 📜 License

This tutorial content is provided for educational purposes.

---

## 🙏 Acknowledgments

This tutorial series is based on comprehensive Kubernetes and cloud-native best practices, designed to provide hands-on production experience for aspiring DevOps engineers and cloud architects.

---

## 🚀 Ready to Begin?

**Start with [Tutorial 1: Introduction & Project Overview](01-introduction-and-project-overview.md)**

May your pods always be healthy and your deployments always successful! 🎊

---

**Last Updated:** December 2024
**Tutorial Version:** 1.0
**Kubernetes Version:** 1.31
**EKS Auto Mode:** Latest
