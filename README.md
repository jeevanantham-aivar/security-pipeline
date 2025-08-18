

[![Security Scan](https://github.com/OWNER/REPO/actions/workflows/security-scan.yml/badge.svg)](https://github.com/OWNER/REPO/actions/workflows/security-scan.yml)

## Project Overview
This repository contains both the application code (`app/`) and Infrastructure-as-Code (`Terraform/`). It uses GitHub Actions to run automated security scanning with Checkov and Bandit on every change, enforcing secure IaC before merge.

## Folder Structure
```
repo-root/
├── app/                         # Application source code (frontend/backend, Dockerfiles, etc.)
├── Terraform/                   # All Infrastructure-as-Code (IaC) using Terraform
│   ├── environments/            # Environment-specific configurations
│   │   ├── dev/
│   │   │   └── terraform.tfvars
│   │   ├── staging/
│   │   │   └── terraform.tfvars
│   │   └── prod/
│   │       └── terraform.tfvars
│   ├── modules/                 # Reusable Terraform modules
│   │   ├── vpc/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   ├── ec2/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   └── ... (other infra modules as needed)
│   ├── main.tf                  # Root module orchestration
│   ├── variables.tf
│   ├── outputs.tf
│   ├── README.md
├── .github/
│   └── workflows/
│       └── security-scan.yml    # GitHub Actions CI/CD & security checks
└── README.md
```

### Folder Purpose
- **app**: Application services, Dockerfiles, manifests, etc.
- **Terraform**: Root Terraform config, reusable modules, and per-environment `.tfvars`.
- **.github/workflows**: CI pipelines; `security-scan.yml` runs Checkov and posts results to PRs.

## Terraform Environments
- Environments: `Terraform/environments/dev|staging|prod/terraform.tfvars`
- `.tfvars` store environment-specific values (e.g., region, AMI, instance type, tags).
- Run locally with:
  - `terraform plan -var-file=Terraform/environments/dev/terraform.tfvars`
  - Swap `dev` with `staging` or `prod` as needed.

## Branching Strategy (Git Flow)
- feature/* → PR into `develop`

- `develop` → merged into `stage`
- `stage` → merged into `main`

Notes:
- Create `develop`, `stage`, and `main` manually at repo initialization.
- Protect `stage` and `main` to require passing checks before merge.

## AWS Authentication
GitHub Actions uses OpenID Connect (OIDC) to authenticate with AWS securely, without storing long‑lived credentials. For the workflow to assume the IAM role `arn:aws:iam::302263040839:role/Githubactions` via OIDC, you must set up trust between GitHub and AWS:
- The IAM role (Githubactions) must trust the GitHub OIDC provider `token.actions.githubusercontent.com`.
- The role’s trust policy must allow this specific GitHub repository (`<owner>/<repo>`) via the OIDC claims (e.g., `sub`, `repository`).
- Without this configuration, the workflow cannot authenticate with AWS and Terraform plan will fail.

High-level steps:
1) Ensure an IAM OIDC identity provider exists for `token.actions.githubusercontent.com`.
2) Update the trust policy of `arn:aws:iam::302263040839:role/Githubactions` to permit your repo.
3) Use `aws-actions/configure-aws-credentials` in the workflow to assume the role via OIDC.

## Security Scanning Workflow (security-scan.yml)
- **Runs Checkov** against Terraform code.
- **Triggers** on both `push` and `pull_request`.
- **Blocks insecure IaC** configurations before merging.

### Trigger Rules
Runs on pull requests in alignment with Git Flow:
- feature/* → `develop`
- feature/* → `main`
- `develop` → `stage`
- `stage` → `main`

This ensures scans run at the same stages as your branching strategy.

### Set Environment Based on Branch
Maps PR source/target to `.tfvars`:
- feature/* → develop → `Terraform/environments/dev/terraform.tfvars`
- feature/* → main → `Terraform/environments/dev/terraform.tfvars`
- develop → stage → `Terraform/environments/staging/terraform.tfvars`
- stage → main → `Terraform/environments/prod/terraform.tfvars`

### Terraform Validation
The workflow:
1) Runs `terraform init`
2) Runs `terraform plan` with the correct `-var-file`
3) Converts the plan to JSON for Checkov to scan against real values:
   - `terraform show -json tfplan.out > tfplan.json`

### Artifacts & PR Feedback
- Uploads CI artifacts:
  - `bandit-report.json` (if app SAST is run)
  - `checkov-report.json`
  - `report.md` (scan summary)
- Posts a sticky PR comment summarizing scan results.

### Example: Local Scan
```bash
cd Terraform
terraform init
terraform plan -out=tfplan.out -var-file=environments/dev/terraform.tfvars
terraform show -json tfplan.out > tfplan.json
checkov -f tfplan.json -o json --output-file-path ../checkov-report.json
```

## CI/CD Workflow
- On PRs, `security-scan.yml` runs and comments results.
- Merges to protected branches are blocked if security checks fail.
- This enforces compliance and prevents insecure Terraform changes from being merged.

## How to Use
1) Clone repo:
```bash
git clone https://github.com/OWNER/REPO.git
cd REPO
```
2) Create feature branch:
```bash
git checkout -b feature/my-change
```
3) Commit & push:
```bash
git add .
git commit -m "feat: my change"
git push -u origin feature/my-change
```
4) Open PR into `develop` → GitHub Actions runs `security-scan.yml`
5) After approval:
- `develop` → `stage` (pre-release)
- `stage` → `main` (production)

## Prerequisites
- Terraform installed
- Checkov installed locally (optional)
- GitHub Actions pre-configured (`.github/workflows/security-scan.yml`) 
=======
## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch**
3. **Make your changes**
4. **Test locally with multiple environments**
5. **Submit a pull request**

### Development Guidelines
- Follow Terraform best practices
- Use consistent naming conventions
- Add comprehensive comments
- Test with all environments
- Ensure Checkov policies pass
=======


=======

