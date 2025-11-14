# Topic 2: Configuring dbt Git Connections

## Overview

Git integration is essential for collaborative dbt development, version control, and CI/CD workflows. This topic covers how to connect dbt Cloud to your Git repository and configure integrations with various Git providers.

## Key Concepts and Definitions

### Git Repository Integration

**Git integration** in dbt Cloud enables:

- **Version control**: Track changes to dbt code over time
- **Collaboration**: Multiple developers working on the same project
- **Code review**: Pull request workflows for quality control
- **CI/CD**: Automated testing and deployment
- **Audit trail**: Complete history of who changed what and when

### Supported Git Providers

dbt Cloud integrates with major Git platforms:

1. **GitHub** (Cloud and Enterprise Server)
2. **GitLab** (Cloud and Self-Managed)
3. **Azure DevOps**
4. **Bitbucket** (Cloud and Server)

### Git Connection Types

#### Native Integrations

Built-in OAuth-based integrations with deep platform features:

- Automatic PR status updates
- Commit status checks
- Direct repository access
- Webhook support for CI triggers

#### Git Clone (Import via HTTPS)

Simpler integration using HTTPS URLs:

- Requires deploy key or access token
- Limited automation features
- Useful for self-hosted Git servers
- Supports any Git-compatible platform

### Repository Structure Requirements

dbt Cloud expects a specific repository structure:

```plaintext
your-dbt-repo/
├── dbt_project.yml          # Required: dbt project configuration
├── models/                   # Your data models
├── macros/                   # Custom macros
├── tests/                    # Data tests
├── analyses/                 # Ad-hoc analyses
├── seeds/                    # CSV files
└── snapshots/                # Snapshot definitions
```

## Practical Examples

### Example 1: Connecting GitHub (Native Integration)

#### Step 1: Install dbt Cloud GitHub App

```markdown
1. Navigate to Account Settings → Integrations
2. Click "Connect GitHub Account"
3. Authorize the dbt Cloud GitHub App
4. Select repositories to grant access
```

#### Step 2: Link Repository to Project

```markdown
Project Settings → Repository
1. Select "GitHub" as repository type
2. Choose your organization
3. Select the repository
4. Click "Save"
```

#### Step 3: Configure Git Clone Strategy

```yaml
# Default subdirectory (if dbt project is not in root)
Subdirectory: /path/to/dbt/project

# Example: Monorepo structure
Repository root: company-analytics-monorepo
dbt Project Path: /dbt/transformations
```

#### Step 4: Test Connection

```markdown
1. Go to IDE
2. Initialize repository
3. Create a new branch
4. Make a change and commit
5. Verify commit appears in GitHub
```

### Example 2: Connecting GitLab with Deploy Token

#### Step 1: Generate GitLab Deploy Token

```markdown
In GitLab:
1. Navigate to Project → Settings → Repository
2. Expand "Deploy Tokens"
3. Create new token:
   - Name: dbt-cloud-access
   - Scopes: read_repository, write_repository
   - Expiration: [Set according to security policy]
4. Copy token (shown only once!)
```

#### Step 2: Configure in dbt Cloud (GitLab)

```markdown
Project Settings → Repository
1. Repository Type: GitLab
2. Repository URL: https://gitlab.com/your-org/your-repo.git
3. Deploy Token Username: gitlab-ci-token (or your token name)
4. Deploy Token: [Paste token from Step 1]
5. Git Clone Strategy: Deploy Token
```

#### Step 3: Verify Access

```bash
# dbt Cloud will attempt to clone:
git clone https://gitlab-ci-token:[TOKEN]@gitlab.com/your-org/your-repo.git

# Verify you see:
✓ Successfully connected to repository
✓ Found dbt_project.yml
```

### Example 3: Connecting Azure DevOps

#### Step 1: Generate Personal Access Token (PAT)

```markdown
In Azure DevOps:
1. Click User Settings → Personal Access Tokens
2. Create new token:
   - Name: dbt-cloud-integration
   - Organization: [Your org]
   - Scopes:
     ✓ Code (Read & Write)
     ✓ Build (Read & Execute)
   - Expiration: [Set according to policy]
3. Copy token value
```

#### Step 2: Configure in dbt Cloud (Azure DevOps)

```markdown
Project Settings → Repository
1. Repository Type: Azure DevOps
2. Organization: your-org-name
3. Project: your-project-name
4. Repository: your-repo-name
5. Authentication: Personal Access Token
6. Token: [Paste PAT from Step 1]
```

#### Step 3: Configure Webhooks (Optional for CI)

```markdown
In Azure DevOps:
1. Project Settings → Service Hooks
2. Create new subscription
3. Select "Web Hooks"
4. Trigger: Pull Request Updated
5. URL: https://cloud.getdbt.com/api/v2/webhooks/azure_devops
6. Add custom header:
   Authorization: Token [your-dbt-cloud-token]
```

### Example 4: Managing Git Settings via API

```bash
# Get current Git connection details
curl -X GET \
  "https://cloud.getdbt.com/api/v3/accounts/{account_id}/projects/{project_id}/repository/" \
  -H "Authorization: Token {api_token}"

# Response
{
  "data": {
    "id": 12345,
    "account_id": 1000,
    "project_id": 5000,
    "repository_url": "https://github.com/your-org/your-repo",
    "git_provider": "github",
    "git_clone_strategy": "github_app",
    "repository_credentials": "***",
    "subdirectory": null
  }
}
```

```bash
# Update repository configuration
curl -X POST \
  "https://cloud.getdbt.com/api/v3/accounts/{account_id}/projects/{project_id}/repository/" \
  -H "Authorization: Token {api_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "git_clone_url": "https://github.com/your-org/your-repo.git",
    "git_provider": "github",
    "subdirectory": "/analytics/dbt"
  }'
```

## Best Practices

### 1. Use Native Integrations When Possible

**Advantages of native integrations:**

- Automatic PR status updates
- Better security with OAuth
- Webhook-triggered CI jobs
- No token management required
- Better user experience

**When to use Git Clone (HTTPS):**

- Self-hosted Git servers not supported by native integrations
- Organizational security policies require token-based access
- Need to integrate with Git-compatible systems (Gitea, Gogs, etc.)

### 2. Implement Branch Protection Rules

```yaml
# GitHub Branch Protection Example
Branch: main

Required status checks:
  ✓ dbt Cloud CI Job

Required reviews:
  ✓ Require pull request reviews before merging
  ✓ Require review from code owners
  ✓ Dismiss stale pull request approvals

Restrictions:
  ✓ Restrict who can push to matching branches
  ✓ Allow force pushes: Disabled
  ✓ Allow deletions: Disabled
```

### 3. Structure Your Repository Appropriately

#### Single Project Structure (Recommended for most teams)

```plaintext
dbt-analytics/
├── .gitignore
├── README.md
├── dbt_project.yml
├── packages.yml
├── models/
│   ├── staging/
│   ├── intermediate/
│   └── marts/
├── macros/
├── tests/
└── snapshots/
```

#### Monorepo Structure (For complex organizations)

```plaintext
company-data-platform/
├── dbt/
│   ├── project_a/
│   │   └── dbt_project.yml
│   ├── project_b/
│   │   └── dbt_project.yml
├── airflow/
├── docs/
└── infrastructure/
```

**Configuration for monorepo:**

```markdown
Project A Settings:
  Repository: company-data-platform
  Subdirectory: /dbt/project_a

Project B Settings:
  Repository: company-data-platform
  Subdirectory: /dbt/project_b
```

### 4. Manage Repository Secrets Properly

**❌ Never commit to Git:**

```yaml
# profiles.yml - DO NOT COMMIT
production:
  outputs:
    prod:
      type: snowflake
      account: xy12345
      password: "secret123"  # NEVER COMMIT SECRETS
```

**✅ Use environment variables in dbt Cloud:**

```yaml
# profiles.yml - Safe to commit
production:
  outputs:
    prod:
      type: snowflake
      account: "{{ env_var('DBT_SNOWFLAKE_ACCOUNT') }}"
      # Password managed in dbt Cloud environment settings
```

### 5. Configure .gitignore Properly

```gitignore
# dbt Cloud .gitignore template
target/
dbt_packages/
logs/
*.pyc
.DS_Store
.env
profiles.yml  # If it contains local development credentials

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
Thumbs.db

# Python virtual environments
venv/
.venv/
```

### 6. Regular Token Rotation

```markdown
Token Rotation Schedule:
- Deploy tokens: Every 90 days
- Personal access tokens: Every 90 days
- Review and update tokens during security audits

Process:
1. Generate new token in Git provider
2. Update in dbt Cloud
3. Test connection
4. Revoke old token
5. Document rotation date
```

### 7. Use CODEOWNERS for Governance

```plaintext
# .github/CODEOWNERS
# Analytics team owns all dbt models
/models/ @your-org/analytics-team

# Data engineering owns sources and staging
/models/staging/ @your-org/data-engineering

# BI team owns marts
/models/marts/ @your-org/bi-team

# Admins approve infrastructure changes
/dbt_project.yml @your-org/data-admins
/packages.yml @your-org/data-admins
```

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Store Credentials in Repository

```yaml
# BAD: Committing credentials
# config/production.yml
snowflake:
  account: xy12345.us-east-1
  user: prod_user
  password: SuperSecret123!  # NEVER DO THIS
  database: PROD
```

**Why it's bad:**

- Credentials exposed in Git history
- Visible to anyone with repository access
- Difficult to rotate without updating code
- Compliance violations

**Solution:** Use dbt Cloud environment variables and connection management

### ❌ DON'T Use Personal Accounts for Production

```markdown
# BAD: Production connected with personal GitHub account
Repository Connection:
  GitHub Account: john.smith@company.com (Personal)
  Access: john.smith's personal token
```

**Why it's bad:**

- Connection breaks when employee leaves
- No separation of personal and work
- Difficult to audit
- Single point of failure

**Solution:** Use organization-level service accounts or GitHub Apps

### ❌ DON'T Ignore Subdirectory Configuration

```markdown
# BAD: Incorrect subdirectory setting
Repository: company-monorepo (root)
dbt Project Location: /analytics/dbt/transform
Subdirectory Setting: [blank]

Result: dbt Cloud looks for dbt_project.yml in root, doesn't find it
```

**Why it's bad:**

- dbt Cloud can't find project files
- IDE won't initialize
- Jobs fail to run

**Solution:** Always configure subdirectory if dbt project is not in repo root

### ❌ DON'T Skip Branch Protection

```markdown
# BAD: No branch protection on main
Settings:
  Branch protection: None
  Anyone can: 
    - Push directly to main
    - Delete main branch
    - Force push
```

**Why it's bad:**

- Accidental overwrites of production code
- No code review process
- Difficult to track who made changes
- Can't enforce CI checks

**Solution:** Implement branch protection with required reviews and status checks

### ❌ DON'T Use Hardcoded Git URLs

```python
# BAD: Hardcoding repository URLs in scripts
def clone_dbt_project():
    repo_url = "https://john:ghp_secrettoken123@github.com/company/dbt.git"
    subprocess.run(["git", "clone", repo_url])
```

**Why it's bad:**

- Tokens in code
- Hard to change
- Security vulnerability
- Not portable

**Solution:** Use environment variables or dbt Cloud API

## Real-World Scenarios and Solutions

### Scenario 1: Migrating from One Git Provider to Another

**Challenge:** Your company is migrating from Bitbucket to GitHub and you need to transition dbt Cloud without disrupting development.

**Solution:**

```markdown
Phase 1: Preparation
1. Create new GitHub repository
2. Mirror Bitbucket repository to GitHub:
   git clone --mirror https://bitbucket.org/company/dbt-project.git
   cd dbt-project.git
   git push --mirror https://github.com/company/dbt-project.git

3. Set up GitHub integration in dbt Cloud (new project or update existing)
4. Configure branch protection and webhooks

Phase 2: Parallel Operations (1 week)
1. Keep both repositories in sync
2. Have team test GitHub integration
3. Update documentation
4. Train team on new workflows

Phase 3: Cutover
1. Announce cutover date
2. Make final sync from Bitbucket to GitHub
3. Update dbt Cloud project to use GitHub
4. Archive Bitbucket repository (read-only)
5. Update all documentation and links

Phase 4: Cleanup (1 week later)
1. Verify all workflows functioning
2. Remove Bitbucket integration from dbt Cloud
3. Delete deploy tokens/access keys
4. Document new procedures
```

### Scenario 2: Multiple dbt Projects in Single Repository

**Challenge:** Your organization has multiple dbt projects in a monorepo and needs to configure separate dbt Cloud projects for each.

**Repository Structure:**

```plaintext
data-platform-monorepo/
├── dbt-projects/
│   ├── finance/
│   │   ├── dbt_project.yml
│   │   └── models/
│   ├── marketing/
│   │   ├── dbt_project.yml
│   │   └── models/
│   └── product/
│       ├── dbt_project.yml
│       └── models/
├── airflow/
└── docs/
```

**Configuration:**

```markdown
dbt Cloud Project: Finance Analytics
  Repository: data-platform-monorepo
  Subdirectory: /dbt-projects/finance
  Custom Branch: main

dbt Cloud Project: Marketing Analytics
  Repository: data-platform-monorepo
  Subdirectory: /dbt-projects/marketing
  Custom Branch: main

dbt Cloud Project: Product Analytics
  Repository: data-platform-monorepo
  Subdirectory: /dbt-projects/product
  Custom Branch: main
```

**CODEOWNERS Configuration:**

```plaintext
# Different teams own different projects
/dbt-projects/finance/ @company/finance-data-team
/dbt-projects/marketing/ @company/marketing-analytics
/dbt-projects/product/ @company/product-data-team
```

### Scenario 3: Self-Hosted GitLab with Custom SSL Certificate

**Challenge:** Your organization uses self-hosted GitLab with a custom SSL certificate, and dbt Cloud can't connect due to certificate validation errors.

**Error Message:**

```text
SSL certificate verification failed: unable to get local issuer certificate
```

**Solution:**

```markdown
Option 1: Add Custom Certificate to dbt Cloud (Enterprise Feature)
1. Contact dbt Cloud support
2. Provide your GitLab instance details
3. Submit custom CA certificate
4. Support team configures certificate trust

Option 2: Use Git Clone with Custom Configuration
1. Generate deploy token in GitLab
2. Use HTTPS clone URL
3. Configure in dbt Cloud:
   Repository URL: https://gitlab.company.internal/team/dbt-project.git
   Deploy Token: [your-token]
   Git Clone Strategy: Deploy Token

Option 3: Configure GitLab with Valid Public Certificate
1. Obtain certificate from public CA (Let's Encrypt, DigiCert, etc.)
2. Install on GitLab server
3. Test with standard SSL verification
4. Connect normally from dbt Cloud
```

### Scenario 4: Webhook Configuration for Advanced CI

**Challenge:** You want CI jobs to trigger automatically on pull requests, but webhooks aren't firing.

**Troubleshooting Steps:**

```markdown
1. Verify Webhook Configuration in Git Provider

GitHub:
  Settings → Webhooks → [dbt Cloud webhook]
  ✓ Payload URL: https://cloud.getdbt.com/api/v2/webhooks/...
  ✓ Content type: application/json
  ✓ Secret: [configured]
  ✓ Events: Pull requests, Pushes

2. Check Recent Deliveries
  - Look for 2xx responses (success)
  - 4xx errors: authentication/authorization issues
  - 5xx errors: dbt Cloud service issues

3. Verify dbt Cloud CI Job Configuration
  - Run on Pull Requests: Enabled
  - Triggered by: Webhook
  - Environment: Development or Staging

4. Test Webhook Manually
curl -X POST \
  https://cloud.getdbt.com/api/v2/webhooks/github \
  -H "Content-Type: application/json" \
  -H "X-Github-Event: pull_request" \
  -d '{
    "action": "opened",
    "pull_request": {
      "number": 123,
      "head": {
        "ref": "feature-branch"
      }
    },
    "repository": {
      "full_name": "your-org/your-repo"
    }
  }'

5. Review dbt Cloud Job Logs
  - Check if webhook was received
  - Verify job trigger conditions met
  - Look for error messages
```

**Common Issues:**

```markdown
Issue: Webhook delivers but CI doesn't trigger
Solution: Check CI job "Triggered by" settings

Issue: 401 Unauthorized responses
Solution: Regenerate webhook secret in both systems

Issue: Webhooks not firing at all
Solution: Verify webhook hasn't been disabled/deleted in Git provider

Issue: CI runs on all commits, not just PRs
Solution: Configure job to run only on "Pull Request" trigger
```

## Sample Exam Questions

### Question 1: Git Provider Selection

**Question:** Your organization uses a self-hosted Git server that is compatible with Git protocols but is not GitHub, GitLab, Azure DevOps, or Bitbucket. Which connection method should you use?

A) Native GitHub integration  
B) Git Clone with HTTPS URL and access token  
C) SSH-based connection  
D) This configuration is not supported by dbt Cloud

**Answer:** B) Git Clone with HTTPS URL and access token

**Explanation:**

- Git Clone (HTTPS) method works with any Git-compatible server
- Native integrations only support specific providers
- SSH connections are not supported in dbt Cloud
- Any Git server supporting HTTPS clone is supported

### Question 2: Repository Structure

**Question:** Your dbt project is located in a subdirectory `/analytics/dbt-transform/` within your repository. Where should you configure this in dbt Cloud?

A) In the dbt_project.yml file  
B) In the Project Settings → Repository → Subdirectory field  
C) In the Environment Settings  
D) This structure is not supported

**Answer:** B) In the Project Settings → Repository → Subdirectory field

**Explanation:**

- The Subdirectory field tells dbt Cloud where to find dbt_project.yml
- This is a repository connection setting, not an environment or project config
- Monorepo structures are fully supported with proper subdirectory configuration

### Question 3: Authentication Token Security

**Question:** A developer's personal access token that was used to connect dbt Cloud to GitLab has expired. Jobs are failing with authentication errors. What is the BEST long-term solution?

A) Ask the developer to generate a new personal token  
B) Switch to using a deploy token tied to the project, not an individual  
C) Disable token expiration in GitLab  
D) Store the token in the repository for easy access

**Answer:** B) Switch to using a deploy token tied to the project, not an individual

**Explanation:**

- Deploy tokens are not tied to individual users, preventing this issue
- Using personal tokens creates dependency on individuals
- Disabling token expiration violates security best practices
- Storing tokens in repositories is a critical security vulnerability

### Question 4: Branch Protection

**Question:** You want to ensure that all changes to your dbt project's main branch go through a pull request review process and pass CI checks. Which of the following should you configure?

A) Only dbt Cloud job settings  
B) Only Git provider branch protection rules  
C) Both Git provider branch protection rules and dbt Cloud CI jobs  
D) Only dbt Cloud environment settings

**Answer:** C) Both Git provider branch protection rules and dbt Cloud CI jobs

**Explanation:**

- Git provider branch protection enforces the review requirement
- dbt Cloud CI jobs provide the checks that must pass
- Both components work together for complete protection
- Environment settings control deployment, not branch protection

### Question 5: Webhook Troubleshooting

**Question:** Your CI job is configured to run on pull requests, but it's not triggering automatically. The webhook shows successful deliveries (200 OK) in GitHub. What should you check next?

A) Git repository connection credentials  
B) CI job "Run on Pull Requests" and "Triggered by Webhook" settings  
C) Data warehouse connection  
D) Deployment environment credentials

**Answer:** B) CI job "Run on Pull Requests" and "Triggered by Webhook" settings

**Explanation:**

- Webhook delivers successfully (200 OK), so connection is fine
- The job must be configured to run on PR events and accept webhook triggers
- Warehouse and deployment credentials wouldn't prevent job triggering
- Repository connection is working if webhook delivers

### Question 6: Multiple Projects Scenario

**Question:** Your company maintains three dbt projects in separate repositories. You want to set up dbt Cloud to manage all three. How many dbt Cloud projects do you need to create?

A) One project with multiple environments  
B) One project with multiple connections  
C) Three separate projects, one for each repository  
D) One project with subdirectory configuration

**Answer:** C) Three separate projects, one for each repository

**Explanation:**

- Each dbt Cloud project connects to one repository (or one subdirectory)
- Multiple environments within a project share the same repository
- Separate repositories require separate dbt Cloud projects
- This enables independent configuration, scheduling, and management

## Summary

Proper Git integration is critical for effective dbt Cloud usage. Key takeaways:

1. **Choose the right integration method:**
   - Native integrations for supported providers
   - Git Clone (HTTPS) for self-hosted or custom servers

2. **Implement security best practices:**
   - Use deploy tokens or service accounts, not personal tokens
   - Regular token rotation
   - Never commit credentials to Git

3. **Configure branch protection:**
   - Require pull request reviews
   - Enforce CI status checks
   - Prevent direct pushes to main

4. **Structure repositories appropriately:**
   - Single project or monorepo patterns
   - Proper .gitignore configuration
   - CODEOWNERS for governance

5. **Test thoroughly:**
   - Verify webhook delivery
   - Test CI job triggering
   - Confirm proper subdirectory configuration

## Next Steps

Continue to [Topic 3: Creating and Maintaining dbt Environments](./03-environments.md) to learn about environment configuration and management.
