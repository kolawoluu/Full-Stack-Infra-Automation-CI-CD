# Production-Grade Terraform & CI/CD

## 🎯 Project Overview

This repository is a showcase of a production-grade Infrastructure as Code (IaC) setup using Terraform and GitHub Actions. It demonstrates how to build, test, and deploy a secure and modular AWS infrastructure while adhering to modern DevOps best practices for automation, governance, and safety.

The core of this project is not just the infrastructure itself, but the sophisticated CI/CD pipeline that manages it. This pipeline is designed to provide high confidence and a strong safety net for every proposed infrastructure change.

## 🏛️ Infrastructure Architecture

The infrastructure is deployed on AWS and consists of a standard three-tier architecture foundation, ready to host containerized applications:

- **Networking**: A custom Virtual Private Cloud (VPC) with public and private subnets across multiple availability zones.
- **Compute**: An Elastic Kubernetes Service (EKS) cluster for container orchestration.
- **Database**: A PostgreSQL database instance using AWS Relational Database Service (RDS).

All components are defined as reusable modules, promoting consistency and maintainability.

## 🔄 The CI/CD Lifecycle: Plan on PR, Apply on Merge

This project implements the industry-standard "Plan on Pull Request, Apply on Merge" strategy, creating a clear separation between proposing a change and enacting it.

### On Pull Request: The "What-If" Analysis
When a pull request is opened to change the infrastructure configuration in `terraform/environments/prod/`, a **`terraform-plan.yml`** workflow triggers to perform a comprehensive, read-only analysis. It answers the question: "What would happen if we merge this?"

- ✅ **`terraform plan`**: Generates a dry-run execution plan.
- ✅ **`tflint` & `tfsec`**: Lints the code and scans for security vulnerabilities.
- ✅ **`infracost`**: Posts a comment on the PR with a detailed breakdown of the monthly cost changes.

No infrastructure is ever changed at this stage.

### On Merge to `main`: The Source of Truth
The `main` branch is the source of truth for the production environment. A merge to `main` signifies that the proposed changes have been reviewed and approved.

- **`terraform-apply.yml`**: This workflow triggers automatically on a push to `main`. Its sole purpose is to run `terraform apply -auto-approve`, reconciling the live infrastructure with the desired state defined in the codebase.

## 📁 Project Structure

The project is organized for clarity and scalability:

```
├── terraform/
│   ├── modules/
│   │   ├── vpc/
│   │   ├── eks/
│   │   └── rds/
│   └── environments/
│       └── prod/
└── .github/
    └── workflows/
        ├── terraform-modules-ci.yml  # Static analysis for modules
        ├── terraform-plan.yml        # PR validation and planning
        └── terraform-apply.yml       # Production deployment on merge
```

For detailed technical documentation, including diagrams and deployment instructions, please see the [Terraform README](./terraform/README.md).

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