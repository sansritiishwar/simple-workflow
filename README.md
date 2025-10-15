# Simple Terraform Workflow

A reusable GitHub Actions workflow that automatically handles Terraform operations across AWS, Azure, and Google Cloud with secure state storage using [tfstate.dev](https://tfstate.dev/).

## 🚀 Features

- **Multi-Cloud Support**: Automatically detects and authenticates with AWS, Azure, and Google Cloud
- **Secure State Storage**: Uses [tfstate.dev](https://tfstate.dev/) for remote state with GitHub token authentication
- **Reusable Workflow**: One central workflow that can be called from any repository
- **Automatic Backend**: Creates tfstate.dev backend configuration dynamically per repository
- **PR Integration**: Shows Terraform plan output in pull request comments
- **Smart Triggers**: Only runs on Terraform file changes

## 📁 Repository Structure

```
.github/
└── workflows/
    ├── terraform.yml           # Original standalone workflow
    ├── reusable-terraform.yml  # Reusable workflow (recommended)
    ├── secret-creation.yml     # Mass secrets deployment workflow
    └── example-terraform-workflow.yml  # Example caller workflow
```

## 🔧 Setup Instructions (Two Options)

### Option 1: Reusable Workflow (Recommended)

This is the **best approach** - maintain one central workflow that all repositories can use.

#### Step 1: Central Workflow Repository
This repository (`sansritiishwar/simple-workflow`) contains the reusable workflow.

#### Step 2: Add to Any Repository
In any repository that needs Terraform automation, create `.github/workflows/terraform.yml`:

```yaml
name: Terraform

on:
  push:
    branches: [ main, master ]
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'
  pull_request:
    branches: [ main, master ]
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'
  workflow_dispatch:

permissions:
  contents: read
  pull-requests: write
  id-token: write

jobs:
  terraform:
    uses: sansritiishwar/simple-workflow/.github/workflows/reusable-terraform.yml@main
    with:
      terraform_version: 'latest'
      working_directory: '.'
    secrets: inherit
```

### Option 2: Repository Template

Alternative approach using GitHub repository templates:

1. **Make this repository a template**:
   - Go to this repository settings
   - Check "Template repository"
   - Save changes

2. **Create new repositories from template**:
   - When creating a new repository, select "Repository template"
   - Choose `sansritiishwar/simple-workflow`
   - The new repository will include all workflow files

## ✨ Why Reusable Workflow is Better

**Reusable Workflow Benefits:**
- ✅ **Single source of truth** - update once, benefits all repositories
- ✅ **Smaller repository footprint** - only 19 lines vs 196 lines
- ✅ **Easier maintenance** - fix bugs in one place
- ✅ **Version control** - can pin to specific versions (@main, @v1.0, etc.)

**Template Repository:**
- ❌ **Duplicated code** - each repository has a full copy
- ❌ **Hard to update** - must manually update each repository
- ✅ **Full control** - can customize per repository if needed

## 🔧 Repository Secrets Configuration (For User Profile)

⚡ **Since this is a user profile (not an organization), configure secrets in this repository and they will be distributed to all your other repositories.**

### Quick Setup: Repository-Level Secrets

1. **Navigate to Repository Settings**:
   - Go to: `https://github.com/sansritiishwar/simple-workflow/settings/secrets/actions`
   - Or: This Repository → Settings → Secrets and variables → Actions

2. **Add these secrets once in this repository**:

Configure the following secrets in **this repository's settings** for distribution across all your repositories:

#### Required Secrets (at minimum one cloud provider)

**AWS Secrets:**
- `AWS_ROLE_ARN`: ARN of the IAM role to assume (e.g., `arn:aws:iam::123456789012:role/GitHubActionsRole`)
- `AWS_REGION`: AWS region (optional, defaults to `us-east-1`)

**Azure Secrets:**
- `AZURE_CREDENTIALS`: Service principal credentials JSON:
  ```json
  {
    "clientId": "your-client-id",
    "clientSecret": "your-client-secret",
    "subscriptionId": "your-subscription-id",
    "tenantId": "your-tenant-id"
  }
  ```

**Google Cloud Secrets:**
- `GCP_CREDENTIALS`: Service account key JSON

#### Optional Secrets
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions (no setup needed)

### 3. Cloud Provider Setup

#### AWS Setup
```bash
# Create IAM role for GitHub Actions
aws iam create-role --role-name GitHubActionsRole --assume-role-policy-document '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR-ACCOUNT:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:sansritiishwar/*:ref:refs/heads/main"
        }
      }
    }
  ]
}'
```

#### Azure Setup
```bash
# Create service principal
az ad sp create-for-rbac --name "GitHubActions" --role contributor --scopes /subscriptions/YOUR-SUBSCRIPTION-ID --sdk-auth
```

#### Google Cloud Setup
```bash
# Create service account
gcloud iam service-accounts create github-actions --display-name="GitHub Actions"

# Create and download key
gcloud iam service-accounts keys create key.json --iam-account=github-actions@YOUR-PROJECT.iam.gserviceaccount.com
```

## 🔄 How It Works

### Automatic Cloud Detection

The workflow automatically detects which cloud provider to authenticate with based on:
- Repository name containing keywords: `aws`, `azure`, `gcp`, `google`
- Commit messages containing keywords: `aws`, `azure`, `gcp`

### State Management with tfstate.dev

The workflow automatically:
1. Creates a backend configuration using tfstate.dev
2. Uses format: `sansritiishwar/{repository-name}` for state storage
3. Authenticates using the GitHub token
4. Provides state locking and encryption

### Workflow Triggers

The workflow runs on:
- Push to `main` or `master` branches (with Terraform file changes)
- Pull requests to `main` or `master` branches (with Terraform file changes)
- Manual trigger via `workflow_dispatch`

### Execution Flow

1. **Setup**: Checkout code, setup Terraform
2. **Authentication**: Configure cloud provider credentials
3. **Backend**: Create tfstate.dev backend configuration
4. **Terraform Operations**:
   - `terraform init` - Initialize with tfstate.dev backend
   - `terraform fmt` - Format check
   - `terraform validate` - Validation
   - `terraform plan` - Create execution plan
   - `terraform apply` - Apply changes (main/master branch only)

## 🚀 Quick Setup for Existing Repositories

To quickly add Terraform automation to your existing repositories (`test1`, `test2`, `test3`, `test4`, `gcp-gke-cluster-infra`):

### Copy this file to each repository:

**File**: `.github/workflows/terraform.yml`
```yaml
name: Terraform

on:
  push:
    branches: [ main, master ]
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'
  pull_request:
    branches: [ main, master ]
    paths:
      - '**.tf'  
      - '.github/workflows/terraform.yml'
  workflow_dispatch:

permissions:
  contents: read
  pull-requests: write
  id-token: write

jobs:
  terraform:
    uses: sansritiishwar/simple-workflow/.github/workflows/reusable-terraform.yml@main
    with:
      terraform_version: 'latest'
      working_directory: '.'
    secrets: inherit
```

That's it! Just **15 lines** vs copying the full 196-line workflow.

### 🔑 Benefits of `secrets: inherit`

- ✅ **Zero manual secret setup** per repository
- ✅ **Automatic inheritance** from organization-level secrets  
- ✅ **Simpler workflow files** - no need to explicitly map each secret
- ✅ **Maintenance-free** - add new secrets at org level, all repos get them

## 📝 Usage Examples

### Basic Terraform Configuration

```hcl
# main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = "my-terraform-state-bucket-${random_id.bucket_suffix.hex}"
}

resource "random_id" "bucket_suffix" {
  byte_length = 8
}
```

The workflow will automatically:
- Detect this is an AWS configuration
- Authenticate with AWS using the configured role
- Store state in tfstate.dev at `amaranath-nagappa/{your-repo-name}`
- Run plan on PRs and apply on main branch

### Multi-Cloud Configuration

```hcl
# multi-cloud.tf
provider "aws" {
  region = "us-east-1"
}

provider "azurerm" {
  features {}
}

provider "google" {
  project = "my-gcp-project"
  region  = "us-central1"
}
```

The workflow will authenticate with all configured cloud providers.

## 🔐 Security Features

- **No Token Storage**: GitHub tokens are not stored by tfstate.dev
- **Encrypted State**: State files are encrypted in Amazon S3 using KMS
- **State Locking**: Prevents concurrent modifications
- **OIDC Authentication**: Uses GitHub's OIDC provider for AWS authentication
- **Scoped Permissions**: Minimal required permissions for each cloud provider

## 🔐 Mass Secrets Deployment

### Overview
The `secret-creation.yml` workflow enables you to deploy secrets across all repositories in your GitHub profile with a single click. This is particularly useful when you have 100+ repositories that need the same secrets for CI/CD operations.

### 🚀 Quick Start

1. **Configure the Personal Access Token**:
   ```bash
   # Create a Personal Access Token with these permissions:
   # - repo (full control) - REQUIRED for accessing private repositories
   # - read:user (optional) - For user profile information
   # Note: Since this is a user profile (not organization), 
   # admin:org permission is not needed
   
   # ⚠️ CRITICAL: The 'repo' scope is essential for private repositories!
   # Without it, the workflow will find 0 repositories.
   ```
   
2. **Add the token to repository secrets**:
   - Go to repository Settings → Secrets and variables → Actions
   - Add `PERSONAL_ACCESS_TOKEN` with your GitHub token

3. **Set up repository secrets** (in this workflow repository):
   Navigate to this repository's Settings → Secrets and variables → Actions and add:
   - `GCP_CREDENTIALS`: Your Google Cloud service account key JSON
   - `AWS_ROLE_ARN`: AWS IAM role ARN for GitHub Actions
   - `AWS_REGION`: AWS region (optional, defaults to us-east-1)
   - `AZURE_CREDENTIALS`: Azure service principal credentials JSON
   - Additional secrets as needed

### 🎯 Usage

#### Automatic Scheduled Runs
The workflow runs automatically **every 5 minutes** in **DRY RUN mode** to monitor for changes without making modifications.

#### Manual Trigger Options
Navigate to Actions → Mass Secrets Deployment → Run workflow:

- **Dry Run**: `true` (recommended first run) - Shows what would be done without making changes
- **Repository Filter**: 
  - `all` - All repositories (public + private)
  - `public` - Only public repositories  
  - `private` - Only private repositories
  - `specific` - Comma-separated list of specific repositories
- **Specific Repos**: List repositories when using "specific" filter (e.g., `repo1,repo2,repo3`)
- **Secrets to Create**: Comma-separated list of secrets (default: `GCP_CREDENTIALS,AWS_ROLE_ARN,AWS_REGION,AZURE_CREDENTIALS`)

#### Example Scenarios

**Scenario 1: Deploy GCP credentials to all repositories**
```yaml
dry_run: false
repository_filter: all
secrets_to_create: GCP_CREDENTIALS
```

**Scenario 2: Update AWS secrets for specific repositories**
```yaml
dry_run: false
repository_filter: specific
specific_repos: my-aws-app,terraform-infra,production-api
secrets_to_create: AWS_ROLE_ARN,AWS_REGION
```

**Scenario 3: Test deployment (dry run)**
```yaml
dry_run: true
repository_filter: all
secrets_to_create: GCP_CREDENTIALS,AWS_ROLE_ARN,AZURE_CREDENTIALS
```

### ⚙️ Workflow Features

- **✅ Batch Processing**: Processes repositories in batches of 10 to respect API rate limits
- **✅ Parallel Execution**: Runs up to 3 batches simultaneously for faster processing
- **✅ Error Handling**: Continues processing even if individual repositories fail
- **✅ Security**: All secrets are encrypted using GitHub's public key encryption
- **✅ Smart Filtering**: Skip archived and disabled repositories automatically  
- **✅ Comprehensive Logging**: Detailed logs for troubleshooting and verification

### 🛡️ Security Considerations

- **Encryption**: All secrets are encrypted before transmission using GitHub's repository-specific public keys
- **Permissions**: Requires `PERSONAL_ACCESS_TOKEN` with appropriate permissions
- **Rate Limiting**: Built-in delays and batch processing to respect GitHub API limits
- **Audit Trail**: Complete workflow logs for compliance and debugging

### 📊 Supported Secrets

Default secrets supported out-of-the-box:
- `GCP_CREDENTIALS` - Google Cloud service account key
- `AWS_ROLE_ARN` - AWS IAM role for GitHub OIDC
- `AWS_REGION` - AWS region
- `AZURE_CREDENTIALS` - Azure service principal credentials  
- `TERRAFORM_API_TOKEN` - Terraform Cloud API token
- `CLOUDFLARE_API_TOKEN` - Cloudflare API token
- `DOCKER_HUB_TOKEN` - Docker Hub access token
- `NPM_TOKEN` - NPM registry token

### 🚨 Important Notes & Limitations

1. **Always test with dry run first** before live deployment
2. **Repository permissions**: The workflow will only process repositories the token has access to
3. **Secret values**: Make sure secrets are configured in this repository before deployment (since this is a user profile, not an organization)
4. **Rate limits**: GitHub API has rate limits - the workflow includes delays to handle this
5. **Overwrite behavior**: The workflow will overwrite existing secrets with the same name
6. **User Profile**: This works with user profiles - no organization setup required

#### ⚠️ Scheduled Workflow Limitations (Every 5min)
- **GitHub Personal Accounts**: Scheduled workflows have severe limitations on private repositories in personal accounts [[memory:7248990]]
- **May not trigger reliably** or at all for private repositories
- **Scheduled runs default to DRY RUN** to prevent accidental deployments
- **Consider alternatives**:
  - Make repository public for reliable scheduling
  - Upgrade to GitHub Pro/Team for better scheduled workflow support  
  - Use manual `workflow_dispatch` triggers instead
  - Use `push` or `repository_dispatch` events as alternatives

## 🚨 Troubleshooting

### Common Issues

1. **Finding 0 Repositories** (Most Common)
   - ❌ **Problem**: Personal Access Token missing or incorrect scopes
   - ✅ **Solution**: Ensure `PERSONAL_ACCESS_TOKEN` has `repo` (full control) scope
   - 📋 **Check**: Go to GitHub → Settings → Developer settings → Personal access tokens
   - 🔍 **Verify**: Token should show "Full control of private repositories" permission

2. **Authentication Failed**
   - Verify secrets are correctly configured
   - Check cloud provider permissions
   - Ensure OIDC provider is set up for AWS

3. **State Backend Issues**
   - Verify GitHub token has repo access
   - Check repository name format
   - Ensure tfstate.dev service is accessible

4. **Workflow Not Triggering**
   - Verify `.tf` files are present
   - Check workflow file is in correct location
   - Ensure branch names match triggers

### Debug Mode

Add this to your Terraform configuration for verbose logging:
```bash
export TF_LOG=DEBUG
```

## 🎯 Best Practices

1. **Repository Naming**: Include cloud provider in repo name for automatic detection
2. **Branch Protection**: Require PR reviews before merging to main/master
3. **State Separation**: Use separate repositories for different environments
4. **Secrets Management**: Store secrets at organization level for reuse
5. **Resource Naming**: Use consistent naming conventions across all resources

## 📚 References

- [tfstate.dev Documentation](https://tfstate.dev/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Terraform Backend Configuration](https://www.terraform.io/docs/language/settings/backends/http.html)
- [GitHub OIDC with AWS](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

---

**User Profile**: sansritiishwar  
**Maintained by**: [sansritiishwar](https://github.com/sansritiishwar)
