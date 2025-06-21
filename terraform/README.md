# Infrastructure as Code - Terraform Setup

## 🏗️ Overview

This directory contains the complete Infrastructure as Code (IaC) setup for the Full-Stack Infrastructure Automation & CI/CD project. The infrastructure is built using Terraform and deployed on AWS, featuring a production-ready architecture with proper security, scalability, and monitoring capabilities.

## 🔄 CI/CD Workflows

This project includes a sophisticated, multi-faceted CI process that runs on every pull request to ensure code quality, security, compliance, and cost-effectiveness. The pipeline is split into two distinct, targeted workflows:

### 1. Module Development Workflow (`terraform-modules-ci.yml`)
This workflow focuses on the quality and security of the reusable Terraform modules. It runs whenever changes are made to the `terraform/modules/` directory.

- **`terraform fmt`**: Ensures consistent code formatting.
- **`tflint`**: Lints the code for best practices, provider-specific issues, and potential errors.
- **`tfsec`**: Scans the code for security vulnerabilities and misconfigurations.

### 2. Environment Deployment Workflow (`terraform-environments-ci.yml`)
This workflow validates the end-to-end deployment for a specific environment (e.g., `prod`). It runs whenever changes are made to the `terraform/environments/` directory.

- **`terraform fmt`**: Ensures consistent code formatting.
- **`terraform init & validate`**: Checks for syntax errors and ensures the configuration is valid.
- **`terraform plan`**: Performs a dry run to generate a speculative execution plan. This step requires AWS credentials.
- **`infracost`**: Analyzes the Terraform plan to provide a detailed cost breakdown, which is posted as a PR comment. This prevents budget overruns.
- **`tflint`**: Lints the environment-specific configuration.
- **`tfsec`**: Scans the final plan for any security issues before deployment.

### Required Status Checks

All checks must pass before a pull request can be merged into `main`, providing a robust quality gate.

## 🏛️ Architecture Overview

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
│  │  │                 │                    │                 │            │   │
│  │  │ ┌─────────────┐ │                    │ ┌─────────────┐ │            │   │
│  │  │ │   Internet  │ │                    │ │   Internet  │ │            │   │
│  │  │ │   Gateway   │ │                    │ │   Gateway   │ │            │   │
│  │  │ └─────────────┘ │                    │ └─────────────┘ │            │   │
│  │  │                 │                    │                 │            │   │
│  │  │ ┌─────────────┐ │                    │ ┌─────────────┐ │            │   │
│  │  │ │   NAT       │ │                    │ │   NAT       │ │            │   │
│  │  │ │  Gateway    │ │                    │ │  Gateway    │ │            │   │
│  │  │ └─────────────┘ │                    │ └─────────────┘ │            │   │
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
│  │  │ │  (Primary)  │ │                    │ │ (Read Rep.) │ │            │   │
│  │  │ └─────────────┘ │                    │ └─────────────┘ │            │   │
│  │  └─────────────────┘                    └─────────────────┘            │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 🔄 Network Communication Flow

### Internet to Application Flow

```
Internet User
    │
    ▼
┌─────────────┐
│   Internet  │
│   Gateway   │
└─────────────┘
    │
    ▼
┌─────────────┐
│   Public    │
│   Subnets   │
│ (AZ-A/B)    │
└─────────────┘
    │
    ▼
┌─────────────┐
│   Private   │
│   Subnets   │
│ (AZ-A/B)    │
└─────────────┘
    │
    ▼
┌─────────────┐
│   EKS Pods  │
│ (Vote/Result)│
└─────────────┘
```

### Application to Database Flow

```
EKS Pods (Vote/Result/Worker)
    │
    ▼
┌─────────────┐
│   Security  │
│   Groups    │
│ (Port 5432) │
└─────────────┘
    │
    ▼
┌─────────────┐
│   RDS       │
│ PostgreSQL  │
│ (Private)   │
└─────────────┘
```

### Internal Service Communication

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Vote      │───▶│   Worker    │───▶│   Result    │
│  Frontend   │    │   Service   │    │  Frontend   │
│             │    │             │    │             │
│ • User      │    │ • Process   │    │ • Display   │
│   Input     │    │   Votes     │    │   Results   │
│ • Send      │    │ • Database  │    │ • Read      │
│   Data      │    │   Updates   │    │   Data      │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ▼
                   ┌─────────────┐
                   │   RDS       │
                   │ PostgreSQL  │
                   │             │
                   │ • Store     │
                   │   Votes     │
                   │ • Retrieve  │
                   │   Results   │
                   └─────────────┘
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

## 📊 Monitoring and Observability

### Infrastructure Metrics

- **VPC**: Subnet utilization, NAT Gateway metrics
- **EKS**: Cluster health, node metrics, pod metrics
- **RDS**: Database performance, connection count, storage usage

### Application Metrics

- **Vote Service**: Request rate, response time, error rate
- **Result Service**: Request rate, response time, error rate
- **Worker Service**: Processing rate, queue depth, error rate

### Security Monitoring

- **Network**: Security group rule changes, unauthorized access attempts
- **IAM**: Role usage, permission changes, access patterns
- **RDS**: Failed login attempts, SSL connections, encryption status

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

**Last Updated**: December 2024  
**Terraform Version**: >= 1.0  
**AWS Provider Version**: ~> 5.0 