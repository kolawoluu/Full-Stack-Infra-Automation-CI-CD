# Full-Stack Infrastructure Automation & CI/CD Pipeline

## 🎯 Project Overview

This project demonstrates a complete DevOps implementation of a 3-tier production-grade infrastructure using the Docker Voting App. It showcases modern DevOps practices including Infrastructure as Code, Kubernetes orchestration, CI/CD automation, monitoring, and security hardening.

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (Vote App)    │◄──►│   (Vote API)    │◄──►│   (PostgreSQL)  │
│   Port: 80      │    │   Port: 5000    │    │   Port: 5432    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes (EKS)                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Ingress   │  │  Prometheus │  │   Grafana   │            │
│  │ Controller  │  │  Monitoring │  │  Dashboards │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AWS Infrastructure                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │     VPC     │  │     EKS     │  │     RDS     │            │
│  │   Subnets   │  │   Cluster   │  │ PostgreSQL  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

## 🛠️ Technology Stack

### Infrastructure
- **Terraform** - Infrastructure as Code
- **AWS** - Cloud provider (VPC, EKS, RDS, EC2)
- **Kubernetes (EKS)** - Container orchestration

### Application
- **Docker** - Containerization
- **Docker Voting App** - Sample application
- **PostgreSQL** - Database

### CI/CD & DevOps
- **GitHub Actions** - CI/CD pipeline
- **Helm** - Kubernetes package manager
- **ArgoCD** - GitOps (Optional)

### Monitoring & Security
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Alertmanager** - Alerting
- **RBAC** - Role-based access control
- **Network Policies** - Security

## 📁 Project Structure

```
├── terraform/                 # Infrastructure as Code
│   ├── modules/
│   │   ├── vpc/              # VPC, subnets, gateways
│   │   ├── eks/              # EKS cluster configuration
│   │   └── rds/              # PostgreSQL database
│   └── environments/
│       └── prod/             # Production environment
├── kubernetes/               # Kubernetes manifests
│   ├── namespaces/           # Namespace definitions
│   ├── deployments/          # Application deployments
│   ├── services/             # Service definitions
│   └── ingress/              # Ingress configurations
├── docker/                   # Docker configurations
│   ├── voting-app/           # Voting application
│   └── docker-compose.yml    # Local development
├── monitoring/               # Monitoring stack
│   ├── prometheus/           # Prometheus configuration
│   └── grafana/              # Grafana dashboards
├── .github/                  # GitHub Actions workflows
│   └── workflows/
└── docs/                     # Documentation
```

## 🚀 Quick Start

### Prerequisites
- AWS CLI configured
- Terraform >= 1.0
- kubectl
- Docker
- Helm

### Deployment Steps

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Full-Stack-Infra-Automation-CI-CD
   ```

2. **Deploy Infrastructure**
   ```bash
   cd terraform/environments/prod
   terraform init
   terraform plan
   terraform apply
   ```

3. **Configure kubectl**
   ```bash
   aws eks update-kubeconfig --region us-west-2 --name voting-app-cluster
   ```

4. **Deploy Application**
   ```bash
   kubectl apply -f kubernetes/namespaces/
   kubectl apply -f kubernetes/deployments/
   kubectl apply -f kubernetes/services/
   kubectl apply -f kubernetes/ingress/
   ```

5. **Deploy Monitoring**
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
   ```

## 📊 Monitoring & Observability

- **Grafana Dashboards**: Application metrics, Kubernetes cluster health
- **Prometheus**: Metrics collection and alerting
- **Alertmanager**: Notifications for critical events

## 🔐 Security Features

- **RBAC**: Role-based access control
- **Network Policies**: Pod-to-pod communication restrictions
- **Secrets Management**: Kubernetes secrets for sensitive data
- **IAM Roles**: Least privilege access for AWS resources

## 🧪 Testing

- **Infrastructure Testing**: Terraform plan/apply validation
- **Application Testing**: Docker container health checks
- **Integration Testing**: End-to-end application flow
- **Security Testing**: Network policy validation

## 📈 CI/CD Pipeline

The GitHub Actions workflow automatically:
1. Lints and tests code
2. Builds Docker images
3. Pushes to container registry
4. Deploys to Kubernetes
5. Runs post-deployment tests

## 🎯 Portfolio Highlights

This project demonstrates:
- ✅ **Infrastructure as Code** with Terraform
- ✅ **Container Orchestration** with Kubernetes
- ✅ **CI/CD Automation** with GitHub Actions
- ✅ **Monitoring & Observability** with Prometheus/Grafana
- ✅ **Security Best Practices** with RBAC and network policies
- ✅ **GitOps** principles (optional with ArgoCD)
- ✅ **Multi-environment** deployment strategy
- ✅ **Production-ready** architecture

## 📝 License

MIT License - see LICENSE file for details

## 🤝 Contributing

This is a portfolio project demonstrating DevOps best practices. Feel free to fork and adapt for your own learning!

---

**Built with ❤️ for DevOps excellence**