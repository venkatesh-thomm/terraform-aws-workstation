# terraform-aws-workstation

Terraform project to provision an AWS EC2 workstation pre-configured with DevOps tooling (Docker, eksctl, kubectl, kubens, k9s, AWS CLI) and automatically creates an EKS cluster on startup.

---

## What It Does

- Launches a **RHEL 9** EC2 instance (`t3.micro`, 50 GB gp3 EBS)
- Opens all inbound/outbound traffic via a dedicated security group
- Runs a user-data bootstrap script that installs:
  - Docker
  - eksctl
  - kubectl
  - kubens
  - k9s
  - AWS CLI (configured for `ec2-user`)
- Clones the `venkatesh-thomm/eksctl` repo and creates an EKS cluster using `eks.yaml`
- Authenticates kubectl against the cluster automatically (`aws eks update-kubeconfig`)
- On `terraform destroy`, automatically deletes the EKS cluster via a remote-exec provisioner

---

## Prerequisites

1. **Terraform** — [Download](https://developer.hashicorp.com/terraform/downloads) and add to `PATH`
   - Verify: `terraform -version`

2. **AWS Account** with permissions to create EC2, Security Groups, and IAM resources

3. **AWS CLI configured** — run `aws configure` on your laptop before applying
   ```powershell
   aws configure
   ```
   Provide your Access Key ID, Secret Access Key, default region (`us-east-1`), and output format (`json`)

4. **eksctl** — install on Windows so you can manage the EKS cluster locally. Run cmd as administrator and run the command
   ```powershell
   choco install eksctl
   ```
   Or download the binary manually from [eksctl releases](https://github.com/eksctl-io/eksctl/releases) and add it to `PATH`
   - Verify: `eksctl version`

---

## Setup

### 1. Clone the repo

```powershell
git clone https://github.com/venkatesh-thomm/terraform-aws-workstation.git
cd terraform-aws-workstation
```

### 2. Configure variables

Create a `terraform.tfvars` file in the project root:

```hcl
aws_access_key = "YOUR_AWS_ACCESS_KEY_ID"
aws_secret_key = "YOUR_AWS_SECRET_ACCESS_KEY"
ssh_password   = "YOUR_EC2_SSH_PASSWORD"
```

> **Security Warning:** Never commit `terraform.tfvars` to version control. It is listed in `.gitignore` — keep it that way.

### 3. Initialize Terraform

```powershell
terraform init
```

### 4. Review the plan

```powershell
terraform plan
```

### 5. Apply

```powershell
terraform apply
```

Type `yes` when prompted. The workstation EC2 instance will be created and the EKS cluster provisioned automatically via user-data.

---

## Destroying the Infrastructure

```powershell
terraform destroy
```

This will:
1. SSH into the workstation and run `eksctl delete cluster` to tear down the EKS cluster
2. Destroy the EC2 instance and security group

---

## Input Variables

| Variable | Description | Default |
|---|---|---|
| `project` | Project name used in resource tags | `roboshop` |
| `environment` | Environment name used in resource tags | `dev` |
| `aws_access_key` | AWS Access Key ID (sensitive) | — |
| `aws_secret_key` | AWS Secret Access Key (sensitive) | — |
| `ssh_password` | SSH password for `ec2-user` (used by destroy provisioner) | — |

---

## Resources Created

| Resource | Description |
|---|---|
| `aws_instance.workstation` | RHEL 9 EC2 instance (t3.micro, 50 GB gp3) |
| `aws_security_group.workstation` | Security group allowing all inbound/outbound traffic |
| `terraform_data.cluster_destroy` | Destroy-time provisioner to delete the EKS cluster |

---

## AMI

Uses the latest `Redhat-9-DevOps-Practice` AMI owned by account `973714476881`, filtered for EBS-backed HVM virtualization.

---

## Provider

| Provider | Version | Region |
|---|---|---|
| `hashicorp/aws` | `6.33.0` | `us-east-1` |
