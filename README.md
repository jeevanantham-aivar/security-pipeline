# Terraform Infrastructure as Code Project

This repository contains a complete Terraform project structure with multi-environment support, CI/CD integration, and security scanning capabilities.

## 🏗️ Project Structure

```
.
├── environments/           # Environment-specific configurations
│   ├── dev/              # Development environment
│   │   └── terraform.tfvars
│   ├── staging/          # Staging environment
│   │   └── terraform.tfvars
│   └── prod/             # Production environment
│       └── terraform.tfvars
├── modules/               # Reusable Terraform modules
│   └── example_module/   # Example EC2 instance module
│       └── main.tf
├── .checkov/             # Checkov security policies
│   └── policies/
│       ├── disallow_public_modules.yaml
│       └── restrict_instance_types.yaml
├── .github/              # GitHub Actions workflows
│   └── workflows/
│       └── terraform-plan.yml
├── main.tf               # Main Terraform configuration
└── README.md             # This file
```

## 🚀 Features

- **Multi-Environment Support**: Separate configurations for dev, staging, and production
- **Modular Architecture**: Reusable Terraform modules
- **CI/CD Integration**: GitHub Actions workflow with matrix-based execution
- **Security Scanning**: Checkov integration with custom policies
- **Cost Optimization**: Enforced instance type restrictions
- **Change Detection**: Automatic detection of which environments need updates

## 🔧 Prerequisites

- Terraform >= 1.0
- AWS CLI configured with appropriate credentials
- GitHub repository with Actions enabled
- Python 3.11+ (for Checkov)

## 📋 Environment Configuration

### Development (dev)
- **Region**: us-east-1
- **Instance Type**: t3.micro
- **Purpose**: Development and testing

### Staging (staging)
- **Region**: us-west-2
- **Instance Type**: t3.small
- **Purpose**: Pre-production testing

### Production (prod)
- **Region**: us-east-1
- **Instance Type**: t3.small
- **Purpose**: Production workloads

## 🏗️ Infrastructure Components

### Example Module
The `example_module` creates:
- EC2 instance with Amazon Linux 2
- Security group with SSH, HTTP, and HTTPS access
- User data script for web server setup
- Proper tagging and resource naming

## 🔒 Security Policies

### Disallow Public Modules (CKV_CUSTOM_001)
- **Severity**: HIGH
- **Purpose**: Ensures only approved private modules are used
- **Allowed Sources**: `app.terraform.io/my-company-org/*`

### Restrict Instance Types (CKV_CUSTOM_002)
- **Severity**: MEDIUM
- **Purpose**: Enforces cost optimization through instance type restrictions
- **Allowed Types**:
  - EC2: t3.micro, t3.small
  - RDS: db.t3.micro, db.t3.small
  - ElastiCache: cache.t3.micro, cache.t3.small
  - Elasticsearch: t3.small.elasticsearch

## 🚀 CI/CD Workflow

### GitHub Actions Workflow: `terraform-plan.yml`

The workflow automatically:
1. **Detects Changes**: Identifies which environment(s) have been modified
2. **Matrix Execution**: Runs Terraform operations in parallel for changed environments
3. **Security Scanning**: Executes Checkov with custom policies
4. **Artifact Storage**: Saves plans and scan results for review

#### Workflow Triggers
- Push to `main` branch
- Pull requests to `main` branch

#### Key Features
- **Smart Change Detection**: Only processes environments with changes
- **Parallel Execution**: Matrix strategy for efficient processing
- **Security First**: Fails on HIGH/CRITICAL vulnerabilities
- **Artifact Management**: Stores results for audit and review

## 🛠️ Usage

### Local Development

1. **Initialize Terraform**:
   ```bash
   terraform init
   ```

2. **Select Environment**:
   ```bash
   # For development
   terraform plan -var-file=environments/dev/terraform.tfvars
   
   # For staging
   terraform plan -var-file=environments/staging/terraform.tfvars
   
   # For production
   terraform plan -var-file=environments/prod/terraform.tfvars
   ```

3. **Apply Changes**:
   ```bash
   terraform apply -var-file=environments/dev/terraform.tfvars
   ```

### CI/CD Pipeline

The GitHub Actions workflow automatically:
- Detects environment changes
- Runs `terraform plan` for affected environments
- Executes Checkov security scanning
- Fails on security violations
- Provides detailed feedback and artifacts

## 🔍 Security Scanning

### Checkov Integration
- **Custom Policies**: Located in `.checkov/policies/`
- **Scan Targets**: Terraform plans and configuration files
- **Failure Threshold**: HIGH and CRITICAL vulnerabilities
- **Output Formats**: CLI, JUnit XML, and text reports

### Policy Categories
1. **Security**: Module source restrictions
2. **Cost Optimization**: Instance type limitations
3. **Compliance**: Resource configuration standards

## 📊 Monitoring and Reporting

### Artifacts Generated
- Terraform plans for each environment
- Checkov security scan results
- JUnit XML reports for CI integration
- Detailed logs and error messages

### Retention Policy
- **Plans**: 30 days
- **Scan Results**: 30 days
- **Logs**: Available in GitHub Actions

## 🚨 Troubleshooting

### Common Issues

1. **Checkov Policy Failures**:
   - Review policy definitions in `.checkov/policies/`
   - Ensure resource configurations meet policy requirements
   - Check policy syntax and validation

2. **Environment Detection Issues**:
   - Verify git history and commit structure
   - Check workflow permissions and branch access
   - Review matrix generation logic

3. **Terraform Validation Errors**:
   - Run `terraform validate` locally
   - Check variable definitions and types
   - Verify module compatibility

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
