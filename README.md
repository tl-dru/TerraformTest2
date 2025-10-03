# TerraformTest

Terraform CI/CD pipeline for Azure infrastructure automation using GitHub Actions.

## Prerequisites

- Azure subscription
- Azure CLI installed
- Terraform installed
- GitHub repository with Actions enabled

## Setup

### 1. Create Azure Service Principal

```bash
az account set --subscription "<your-subscription-name>"
subscriptionId=$(az account show | jq -r '.id')
sp=$(az ad sp create-for-rbac --name "TerraformSP" --role Contributor --scopes /subscriptions/$subscriptionId)
```

### 2. Configure GitHub Secrets

Set the following secrets in your GitHub repository (`Settings` > `Secrets and variables` > `Actions`):

- `ARM_CLIENT_ID` - Service Principal App ID
- `ARM_CLIENT_SECRET` - Service Principal Password
- `ARM_TENANT_ID` - Azure Tenant ID
- `ARM_SUBSCRIPTION_ID` - Azure Subscription ID

**Using GitHub CLI:**

```bash
gh secret set ARM_CLIENT_ID --body "<app-id>"
gh secret set ARM_CLIENT_SECRET --body "<password>"
gh secret set ARM_TENANT_ID --body "<tenant-id>"
gh secret set ARM_SUBSCRIPTION_ID --body "<subscription-id>"
```

## Workflow

The GitHub Actions workflow automatically:

- **On Pull Requests**: Runs `terraform init`, `terraform fmt -check`, and `terraform plan`
- **On Push to main**: Runs `terraform init`, `terraform fmt -check`, `terraform plan`, and `terraform apply`

## Local Development

Set environment variables for local Terraform execution:

```bash
export ARM_CLIENT_ID="<app-id>"
export ARM_CLIENT_SECRET="<password>"
export ARM_TENANT_ID="<tenant-id>"
export ARM_SUBSCRIPTION_ID="<subscription-id>"

terraform init
terraform plan
terraform apply
```

## Documentation

See [CICD with GitHub Actions & DevOps Integration.md](CICD%20with%20GitHub%20Actions%20%26%20DevOps%20Integration.md) for detailed instructions.
