# GitHub Actions Deep Dive

(Due to technical issues, the search service is temporarily unavailable.)

I cannot directly generate a PDF document here, but I can provide you with a **Markdown template** that you can easily convert to PDF using tools like [Pandoc](https://pandoc.org), Google Docs, or Microsoft Word. Below is a structured document that includes the Terraform code, GitHub Actions workflow, and Helm instructions.

---

```markdown
# EKS Deployment with Terraform, Helm, and GitHub Actions

## Introduction
This document provides step-by-step instructions to:
1. Deploy an Amazon EKS cluster using Terraform.
2. Install a Helm chart (e.g., NGINX Ingress Controller).
3. Automate the process using GitHub Actions.

---

## Prerequisites
- AWS account with IAM credentials.
- Terraform installed locally (for testing).
- Helm CLI (optional for local testing).
- GitHub repository with AWS secrets configured:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`

---

## 1. Deploy EKS with Terraform

### Terraform Configuration
Create the following files:

#### `main.tf`
```hcl
provider "aws" {
  region = "us-west-2"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "18.21.0"

  cluster_name    = "my-eks-cluster"
  cluster_version = "1.24"

  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets

  eks_managed_node_groups = {
    default = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1
      instance_types   = ["t3.medium"]
    }
  }
}
```

#### `outputs.tf`
```hcl
output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_name" {
  value = module.eks.cluster_name
}
```

---

## 2. Install Helm Chart

### Helm Values File (`helm_values.yaml`)
```yaml
controller:
  replicaCount: 2
  service:
    type: LoadBalancer
```

### Helm Installation Command
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx -f helm_values.yaml
```

---

## 3. GitHub Actions Automation

### Workflow File (`.github/workflows/deploy-eks.yaml`)
```yaml
name: Deploy EKS and Helm Chart

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Terraform Init & Apply
        run: |
          terraform init
          terraform apply -auto-approve
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Configure AWS CLI and kubectl
        run: |
          aws eks update-kubeconfig --region us-west-2 --name my-eks-cluster

      - name: Install Helm
        run: |
          curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
          chmod 700 get_helm.sh
          ./get_helm.sh

      - name: Deploy Helm Chart
        run: |
          helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
          helm install nginx-ingress ingress-nginx/ingress-nginx -f helm_values.yaml
```

---

## 4. Security Best Practices
1. **AWS Credentials**: Use IAM roles with least privilege.
2. **Secrets Management**: Store sensitive data in GitHub Secrets.
3. **Terraform State**: Use a remote backend (e.g., Amazon S3) with encryption.

---

## 5. Troubleshooting
| Issue                          | Solution                                  |
|--------------------------------|-------------------------------------------|
| Terraform authentication error | Verify AWS credentials in GitHub Secrets. |
| Helm chart installation failed | Check Kubernetes cluster access.         |
| EKS node group not scaling     | Validate IAM roles for autoscaling.       |

---

## Conclusion
This setup automates EKS deployment and Helm chart installation using Terraform and GitHub Actions. Replace placeholder values (e.g., `us-west-2`) with your preferred AWS region.
```

---

