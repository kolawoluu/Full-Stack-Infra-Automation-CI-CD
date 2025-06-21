# Infrastructure as Code - Terraform Setup

## 🏗️ Overview

This directory contains the complete Infrastructure as Code (IaC) setup for the project. The infrastructure is built using Terraform and deployed on AWS, featuring a production-ready architecture with proper security, scalability, and modularity.

## 🔄 CI/CD Workflows

This project includes a sophisticated, multi-faceted CI process that runs on every pull request to ensure code quality, security, compliance, and cost-effectiveness. The pipeline is split into distinct, targeted workflows based on the path of the changes.

### 1. Module Development Workflow (`terraform-modules-ci.yml`)
This workflow focuses on the quality and security of the reusable Terraform modules. It runs whenever changes are made to the `terraform/modules/` directory and performs static analysis, requiring no cloud credentials.

- **`terraform fmt`**: Ensures consistent code formatting.
- **`tflint`**: Lints the code for best practices and potential errors.
- **`tfsec`**: Scans the code for security vulnerabilities.

### 2. Environment Plan Workflow (`terraform-plan.yml`)
This workflow validates an end-to-end deployment plan when a pull request targets a specific environment (e.g., `prod`). It provides a comprehensive, read-only preview of the proposed changes.

- **`terraform plan`**: Performs a dry run to generate a speculative execution plan.
- **`infracost`**: Analyzes the plan to provide a detailed cost breakdown in a PR comment.

### 3. Environment Apply Workflow (`terraform-apply.yml`)
This workflow triggers only when a pull request to an environment branch is merged. It takes the approved plan and applies it to the live AWS environment.

- **`terraform apply`**: Reconciles the live infrastructure with the desired state in the `main` branch.

## 🏛️ Infrastructure Architecture

The AWS infrastructure is designed to provide a secure and scalable foundation for a typical three-tier application.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                AWS Infrastructure                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                              VPC (10.0.0.0/16)                         │   │
│  │                                                                         │   │
│  │  ┌─────────────────┐                    ┌─────────────────┐            │   │
│  │  │   Public AZ-A   │                    │   Public AZ-B   │            │   │
│  │  │  (10.0.1.0/24)  │                    │  (10.0.2.0/24)  │            │   │
│  │  └─────────────────┘                    └─────────────────┘            │   │
│  │                                                                         │   │
│  │  ┌─────────────────┐                    ┌─────────────────┐            │   │
│  │  │  Private AZ-A   │                    │  Private AZ-B   │            │   │
│  │  │ (10.0.11.0/24)  │                    │ (10.0.12.0/24)  │            │   │
│  │  │                 │                    │                 │            │   │
│  │  │ ┌─────────────┐ │                    │ ┌─────────────┐ │            │   │
│  │  │ │   EKS       │ │                    │ │   EKS       │ │            │   │
│  │  │ │  Worker     │ │                    │ │  Worker     │ │            │   │
│  │  │ │   Nodes     │ │                    │ │   Nodes     │ │            │   │
│  │  │ └─────────────┘ │                    │ └─────────────┘ │            │   │
│  │  │                 │                    │                 │            │   │
│  │  │ ┌─────────────┐ │                    │ ┌─────────────┐ │            │   │
│  │  │ │    RDS      │ │                    │ │    RDS      │ │            │   │
│  │  │ │ PostgreSQL  │ │                    │ │ PostgreSQL  │ │            │   │
│  │  │ └─────────────┘ │                    │ └─────────────┘ │            │   │
│  │  └─────────────────┘                    └─────────────────┘            │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 🛠️ Module Structure

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf          # VPC, subnets, gateways, route tables
│   │   ├── variables.tf     # Input variables
│   │   └── outputs.tf       # Output values
│   ├── eks/
│   │   ├── main.tf          # EKS cluster, node groups, IAM roles
│   │   ├── variables.tf     # Input variables
│   │   └── outputs.tf       # Output values
│   └── rds/
│       ├── main.tf          # RDS instance, security groups, monitoring
│       ├── variables.tf     # Input variables
│       └── outputs.tf       # Output values
└── environments/
    └── prod/
        ├── main.tf          # Main configuration
        ├── variables.tf     # Environment variables
        ├── outputs.tf       # Environment outputs
        └── terraform.tfvars.example # Example variable values
```

## 🔐 Security Architecture

### Network Security

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              Security Architecture                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                              Security Groups                            │   │
│  │                                                                         │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │   │
│  │  │   EKS Cluster   │  │   RDS Database  │  │   Load Balancer │         │   │
│  │  │   Security      │  │   Security      │  │   Security      │         │   │
│  │  │   Group         │  │   Group         │  │   Group         │         │   │
│  │  │                 │  │                 │  │                 │         │   │
│  │  │ • Port 443      │  │ • Port 5432     │  │ • Port 80       │         │   │
│  │  │ • VPC CIDR      │  │ • EKS SG Only   │  │ • Port 443      │         │   │
│  │  │ • HTTPS Only    │  │ • SSL/TLS       │  │ • Internet      │         │   │
│  │  │ • API Access    │  │ • Encryption    │  │ • Public        │         │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 🚀 Deployment Instructions

### Prerequisites

1. **AWS CLI** configured with appropriate permissions
2. **Terraform** >= 1.0
3. **S3 bucket** for Terraform state (optional but recommended)

### Step 1: Configure AWS

```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, and region (us-west-2)
```

### Step 2: Set Up Terraform Backend (Optional)

```bash
# Create S3 bucket for Terraform state
aws s3 mb s3://voting-app-terraform-state --region us-west-2
aws s3api put-bucket-versioning --bucket voting-app-terraform-state --versioning-configuration Status=Enabled
```

### Step 3: Configure Variables

```bash
cd terraform/environments/prod
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your specific values
```

### Step 4: Initialize Terraform

```bash
terraform init
```

### Step 5: Plan Deployment

```bash
terraform plan -var="rds_db_password=your-secure-password"
```

### Step 6: Deploy Infrastructure

```bash
terraform apply -var="rds_db_password=your-secure-password"
```

### Step 7: Configure kubectl

```bash
aws eks update-kubeconfig --region us-west-2 --name voting-app-cluster
kubectl get nodes
```

## 🔧 Troubleshooting

### Common Issues

1. **VPC Creation Fails**:
   - Check AWS region availability
   - Verify CIDR block conflicts
   - Ensure sufficient IP addresses

2. **EKS Cluster Creation Fails**:
   - Verify IAM permissions
   - Check subnet configuration
   - Ensure sufficient node capacity

3. **RDS Creation Fails**:
   - Verify subnet group configuration
   - Check security group rules
   - Ensure parameter group compatibility

### Useful Commands

```bash
# Check Terraform state
terraform show
terraform output

# Check AWS resources
aws ec2 describe-vpcs --filters "Name=tag:Project,Values=voting-app"
aws eks describe-cluster --name voting-app-cluster --region us-west-2
aws rds describe-db-instances --db-instance-identifier voting-app-db

# Check Kubernetes resources
kubectl get nodes
kubectl get pods --all-namespaces
kubectl get services --all-namespaces
```

## 🧹 Cleanup

To destroy the infrastructure:

```bash
cd terraform/environments/prod
terraform destroy -var="rds_db_password=your-secure-password"
```

**Warning**: This will permanently delete all resources. Ensure you have backups of any important data.

## 📚 Additional Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [EKS Best Practices](https://aws.github.io/aws-eks-best-practices/)
- [RDS Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- [VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)

---
