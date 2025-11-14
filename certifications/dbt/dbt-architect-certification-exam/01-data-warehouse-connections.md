# Topic 1: Configuring dbt Data Warehouse Connections

## Overview

Configuring data warehouse connections is the foundation of any dbt Cloud implementation. This topic covers how to establish secure, reliable connections between dbt Cloud and your data warehouse, including various authentication methods and security configurations.

## Key Concepts and Definitions

### Data Warehouse Connection

A **data warehouse connection** in dbt Cloud defines how dbt connects to your data platform to execute SQL and manage transformations. This includes:

- **Connection credentials**: Authentication information (username/password, key pairs, OAuth tokens)
- **Network configuration**: IP allowlists, SSL settings, connection timeout
- **Warehouse metadata**: Host URL, port, database/project name, account identifiers
- **Adapter-specific settings**: Warehouse-specific parameters (e.g., BigQuery project, Snowflake warehouse size)

### Supported Data Warehouses

dbt Cloud supports multiple data warehouse platforms:

- **Snowflake**
- **BigQuery**
- **Redshift**
- **Databricks**
- **Postgres**
- **Azure Synapse**
- **Fabric**
- **Starburst/Trino**

### Connection Types

1. **Development Connections**: Used by individual developers in the IDE
2. **Deployment Connections**: Used by jobs running in production or CI/CD environments

### Authentication Methods

#### 1. Username and Password

- Basic authentication method
- Credentials stored encrypted in dbt Cloud
- Suitable for development environments
- Not recommended for production due to credential rotation complexity

#### 2. Key Pair Authentication

- More secure than username/password
- Uses public/private key cryptography
- Recommended for production deployments
- Supports automatic key rotation via API

#### 3. OAuth Authentication

- Modern authentication standard
- Users authenticate through identity provider
- Token-based access with automatic refresh
- Best user experience for IDE access
- Requires OAuth application setup

#### 4. Service Accounts

- Dedicated non-human accounts for production jobs
- Consistent identity across job runs
- Easier auditing and access control
- Required for deployment environments

## Practical Examples

### Example 1: Configuring a Snowflake Connection

#### Step 1: Navigate to Connection Settings

```shell
Account Settings → Projects → [Your Project] → Connection
```

#### Step 2: Enter Connection Details

```yaml
Account: xy12345.us-east-1
Database: ANALYTICS
Warehouse: TRANSFORMING
Role: DBT_CLOUD_ROLE
```

#### Step 3: Configure Authentication

**For Development (OAuth):**

```yaml
Auth Method: OAuth
Allow SSO Login: Enabled
```

**For Deployment (Key Pair):**

```yaml
Auth Method: Key Pair
Username: DBT_CLOUD_SERVICE_ACCOUNT
Private Key: [Upload .pem file or paste key]
Private Key Passphrase: [If encrypted]
```

### Example 2: Configuring IP Allowlist

Many organizations restrict warehouse access by IP address. dbt Cloud publishes a list of IP addresses that must be allowlisted.

#### Snowflake Network Policy Example

```sql
-- Create network policy for dbt Cloud
CREATE NETWORK POLICY dbt_cloud_policy
  ALLOWED_IP_LIST = (
    '52.45.144.63/32',
    '54.81.134.249/32',
    '52.22.161.231/32'
    -- Add all dbt Cloud IPs for your region
  );

-- Apply to service account
ALTER USER dbt_cloud_service_account 
  SET NETWORK_POLICY = dbt_cloud_policy;
```

#### BigQuery Authorized Networks Example

```shell
1. Go to BigQuery → Settings → Authorized Networks
2. Add dbt Cloud IP ranges:
   - 52.45.144.63/32
   - 54.81.134.249/32
   - 52.22.161.231/32
   [Add all IPs for your region]
```

**Important**: dbt Cloud IP addresses vary by region. Always check the [official documentation](https://docs.getdbt.com/docs/cloud/about-cloud/access-regions-ip-addresses) for your specific region.

### Example 3: Setting Up OAuth for BigQuery

#### Step 1: Create OAuth Application in Google Cloud

```bash
# Navigate to Google Cloud Console
# APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID

Application Type: Web application
Authorized redirect URIs: 
  https://cloud.getdbt.com/complete/bigquery
  https://YOUR_ACCOUNT.getdbt.com/complete/bigquery
```

#### Step 2: Configure OAuth in dbt Cloud

```yaml
Project Settings → Connection → OAuth Settings
Client ID: [Your Google OAuth Client ID]
Client Secret: [Your Google OAuth Client Secret]
```

#### Step 3: Developer Authentication Flow

```shell
1. Developer opens dbt Cloud IDE
2. Clicks "Connect to BigQuery"
3. Redirected to Google OAuth consent screen
4. Grants permissions to dbt Cloud
5. Redirected back to dbt Cloud with access token
6. Token stored securely and refreshed automatically
```

### Example 4: Testing a Connection

#### In the UI

```shell
1. Navigate to Project Settings → Connection
2. Scroll to "Test Connection" section
3. Click "Test Connection"
4. Verify success message:
   ✓ Connection test succeeded
   ✓ Database: ANALYTICS
   ✓ Schema: DBT_DEV
```

#### Via API

```bash
curl -X POST \
  https://cloud.getdbt.com/api/v2/accounts/{account_id}/projects/{project_id}/connection/test/ \
  -H 'Authorization: Token {your_api_token}' \
  -H 'Content-Type: application/json'
```

**Expected Response:**

```json
{
  "status": {
    "code": 200,
    "is_success": true,
    "user_message": "Connection test succeeded",
    "developer_message": ""
  },
  "data": {
    "connection_status": "OK",
    "database": "ANALYTICS",
    "schema": "DBT_DEV"
  }
}
```

## Best Practices

### 1. Use OAuth for Development Environments

**Why**: OAuth provides the best security and user experience

- Individual user accountability
- Automatic token refresh
- No credential sharing
- Easy onboarding for new developers

### 2. Use Service Accounts with Key Pairs for Production

**Why**: Service accounts ensure consistent execution

- Dedicated credentials for automation
- Easier to audit and monitor
- Key rotation without affecting multiple users
- Clear separation between dev and prod access

### 3. Implement Principle of Least Privilege

```sql
-- Snowflake example: Create role with minimal permissions
CREATE ROLE DBT_CLOUD_ROLE;

-- Grant only necessary privileges
GRANT USAGE ON WAREHOUSE TRANSFORMING TO ROLE DBT_CLOUD_ROLE;
GRANT USAGE ON DATABASE ANALYTICS TO ROLE DBT_CLOUD_ROLE;
GRANT USAGE ON SCHEMA ANALYTICS.DBT_PROD TO ROLE DBT_CLOUD_ROLE;
GRANT CREATE TABLE ON SCHEMA ANALYTICS.DBT_PROD TO ROLE DBT_CLOUD_ROLE;
GRANT CREATE VIEW ON SCHEMA ANALYTICS.DBT_PROD TO ROLE DBT_CLOUD_ROLE;

-- Do NOT grant:
-- - DROP DATABASE
-- - OWNERSHIP transfer
-- - ACCOUNTADMIN role
```

### 4. Separate Development and Production Credentials

**Development**:

- Individual OAuth accounts
- Access to dev schemas only
- Smaller warehouse sizes

**Production**:

- Dedicated service account
- Access to prod schemas only
- Appropriately sized warehouses
- Different network policies if needed

### 5. Regular Key Rotation

```python
# Example: Rotate Snowflake key pair via API
import requests
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.backends import default_backend

# Generate new key pair
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048,
    backend=default_backend()
)

# Update in dbt Cloud via API
response = requests.post(
    f'https://cloud.getdbt.com/api/v2/accounts/{account_id}/projects/{project_id}/credentials/',
    headers={'Authorization': f'Token {api_token}'},
    json={
        'type': 'snowflake',
        'private_key': private_key.decode('utf-8')
    }
)
```

**Recommended Rotation Schedule**:

- Production keys: Every 90 days
- Service account passwords: Every 60 days
- Review schedule during security audits

### 6. Monitor Connection Health

Set up monitoring to detect connection issues:

- Failed authentication attempts
- Network timeouts
- Permission errors
- Unusual connection patterns

### 7. Document Connection Configuration

Maintain documentation including:

- Warehouse account/project identifiers
- Database and schema names
- Service account names
- IP addresses requiring allowlisting
- OAuth application details
- Key rotation schedule

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Share User Credentials

**Problem**: Multiple developers using the same username/password

```yaml
# BAD: Everyone uses the same account
Username: shared_developer_account
Password: CompanyPassword123!
```

**Why it's bad**:

- No audit trail of who made changes
- Password changes affect everyone
- Security vulnerability
- Violates compliance requirements

**Better approach**: Use OAuth for individual authentication

### ❌ DON'T Use Admin Accounts for dbt

**Problem**: Granting dbt unnecessarily high privileges

```sql
-- BAD: Granting admin role to dbt
GRANT ACCOUNTADMIN TO ROLE DBT_CLOUD_ROLE;
```

**Why it's bad**:

- Security risk if credentials compromised
- dbt could accidentally modify critical infrastructure
- Violates least privilege principle

**Better approach**: Create dedicated role with minimal permissions

### ❌ DON'T Hardcode Credentials in Git

**Problem**: Storing credentials in profiles.yml and committing to Git

```yaml
# BAD: Never commit this!
# profiles.yml
production:
  target: prod
  outputs:
    prod:
      type: snowflake
      account: xy12345
      user: dbt_user
      password: "SuperSecretPassword123!"  # NEVER DO THIS
```

**Why it's bad**:

- Credentials exposed in version control
- Difficult to rotate compromised credentials
- Compliance violations

**Better approach**: Use dbt Cloud credential management or environment variables

### ❌ DON'T Ignore Connection Failures

**Problem**: Continuing to work without properly testing connections

```shell
Connection test failed: Network timeout
[Developer proceeds anyway, thinking it might work later]
```

**Why it's bad**:

- Wastes time debugging later
- May indicate security policy issues
- Could result in partial data loads

**Better approach**: Investigate and resolve connection issues immediately

### ❌ DON'T Use Same Credentials Across Environments

**Problem**: Using production credentials in development

```yaml
# BAD: Same service account for dev and prod
Dev Environment: DBT_PROD_SERVICE_ACCOUNT
Prod Environment: DBT_PROD_SERVICE_ACCOUNT
```

**Why it's bad**:

- Developers could accidentally modify production data
- Harder to track and audit changes
- Increased blast radius for security incidents

**Better approach**: Separate credentials per environment

## Real-World Scenarios and Solutions

### Scenario 1: Multi-Region Deployment

**Challenge**: Your organization has data warehouses in multiple geographic regions (US, EU, APAC) and needs to deploy dbt Cloud to work with all of them.

**Solution**:

```yaml
# Project Structure
Projects:
  - dbt_us_analytics
    Connection: Snowflake US-EAST-1
    Deployment: dbt Cloud North America
    
  - dbt_eu_analytics
    Connection: Snowflake EU-WEST-1
    Deployment: dbt Cloud EMEA
    
  - dbt_apac_analytics
    Connection: Snowflake AP-SOUTHEAST-2
    Deployment: dbt Cloud APAC
```

**Key Considerations**:

- Use dbt Cloud region matching data warehouse region for lower latency
- Configure region-specific IP allowlists
- Ensure OAuth applications configured for each region
- Document timezone differences for scheduling

### Scenario 2: Migration from dbt Core to dbt Cloud

**Challenge**: Your team is migrating from self-hosted dbt Core to dbt Cloud. You need to transition connection management without disrupting daily operations.

**Migration Steps**:

```markdown
Phase 1: Setup (Week 1)
- Create dbt Cloud account
- Configure connection to development warehouse
- Set up OAuth for pilot users
- Test connection thoroughly

Phase 2: Pilot (Week 2-3)
- Migrate 2-3 developers to dbt Cloud
- Validate IDE functionality
- Confirm job execution
- Gather feedback

Phase 3: Full Migration (Week 4-6)
- Migrate all developers
- Configure production deployment credentials
- Set up CI/CD jobs
- Migrate scheduled jobs
- Update documentation

Phase 4: Optimization (Week 7-8)
- Implement service accounts
- Set up monitoring
- Configure key rotation
- Security audit
```

**Critical Success Factor**: Maintain parallel operations until fully validated

### Scenario 3: IP Allowlist Configuration for Enterprise Security

**Challenge**: Your security team requires all external connections to be allowlisted, but dbt Cloud IPs aren't documented in your infrastructure-as-code repository.

**Solution**:

```terraform
# Terraform example for Snowflake
# variables.tf
variable "dbt_cloud_ips" {
  description = "dbt Cloud IP addresses for US region"
  type        = list(string)
  default = [
    "52.45.144.63/32",
    "54.81.134.249/32",
    "52.22.161.231/32",
    # Full list from dbt documentation
  ]
}

# main.tf
resource "snowflake_network_policy" "dbt_cloud" {
  name            = "DBT_CLOUD_POLICY"
  allowed_ip_list = var.dbt_cloud_ips
  comment         = "Network policy for dbt Cloud access"
}

resource "snowflake_user" "dbt_cloud_service" {
  name                 = "DBT_CLOUD_SERVICE"
  default_role         = snowflake_role.dbt_cloud.name
  network_policy       = snowflake_network_policy.dbt_cloud.name
  must_change_password = false
  
  lifecycle {
    ignore_changes = [
      rsa_public_key,  # Managed via dbt Cloud API
    ]
  }
}
```

**Additional Steps**:

1. Create pull request with infrastructure changes
2. Security team reviews and approves
3. Apply changes to Snowflake
4. Test connection from dbt Cloud
5. Document in runbook

### Scenario 4: OAuth Setup Failing

**Challenge**: Developers receive "OAuth authentication failed" error when trying to connect to BigQuery from dbt Cloud IDE.

**Troubleshooting Steps**:

```markdown
1. Check OAuth Application Configuration
   - Verify Client ID matches in Google Cloud Console and dbt Cloud
   - Confirm Client Secret is correct
   - Check redirect URIs are correctly configured

2. Verify Scopes
   Required scopes for BigQuery:
   - https://www.googleapis.com/auth/bigquery
   - https://www.googleapis.com/auth/drive.readonly (if using Google Sheets)

3. Check User Permissions
   - User must have BigQuery Data Editor role
   - Project-level IAM permissions
   - Organization policies allowing OAuth

4. Test OAuth Flow
   - Clear browser cookies/cache
   - Try incognito/private browsing
   - Check browser console for errors
   - Verify network connectivity

5. Common Error Messages
   "redirect_uri_mismatch": Redirect URI not configured
   "invalid_client": Client ID or Secret incorrect
   "access_denied": User doesn't have required permissions
```

**Resolution Example**:

```bash
# Correct OAuth redirect URI format
https://cloud.getdbt.com/complete/bigquery
https://YOUR_ACCOUNT_SLUG.getdbt.com/complete/bigquery

# Not: http:// (must be https://)
# Not: trailing slashes
# Not: additional path segments
```

## Sample Exam Questions

### Question 1: Authentication Methods

**Question**: Your organization requires that all production database access be auditable with individual user attribution and automatic credential rotation. Which authentication method should you use for dbt Cloud deployment connections?

A) Username and password authentication  
B) OAuth with individual user accounts  
C) Service account with key pair authentication  
D) Shared developer account with password

**Answer**: C) Service account with key pair authentication

**Explanation**:

- Service accounts provide consistent identity for production jobs
- Key pairs can be rotated programmatically via API
- While OAuth provides individual attribution, it's designed for development IDE access, not deployment jobs
- Service accounts meet the requirement for "production database access" while OAuth is better for development
- Key pair authentication is more secure than username/password and supports easier rotation

### Question 2: IP Allowlisting

**Question**: You're deploying dbt Cloud in the EU region and need to configure Snowflake network policies. Which of the following is the correct approach?

A) Use the same IP addresses as the US region  
B) Configure all possible dbt Cloud IP addresses from all regions  
C) Use only the IP addresses specified for the EU region in dbt Cloud documentation  
D) IP allowlisting is not necessary for dbt Cloud connections

**Answer**: C) Use only the IP addresses specified for the EU region in dbt Cloud documentation

**Explanation**:

- dbt Cloud uses different IP addresses for different regions
- Using only the necessary IPs follows the principle of least privilege
- Over-allowlisting (option B) creates unnecessary security exposure
- dbt Cloud IPs are region-specific and documented per region

### Question 3: Connection Testing

**Question**: After configuring a new connection in dbt Cloud, the connection test succeeds, but jobs fail with "Permission denied" errors. What is the most likely cause?

A) The connection test doesn't actually validate permissions  
B) The development credentials work, but deployment credentials are not configured correctly  
C) The IP allowlist is missing the dbt Cloud addresses  
D) The database is temporarily unavailable

**Answer**: B) The development credentials work, but deployment credentials are not configured correctly

**Explanation**:

- Connection tests typically use development credentials
- Jobs use deployment environment credentials, which are configured separately
- If IP allowlist was the issue, the connection test would also fail
- A temporarily unavailable database would affect both tests and jobs

### Question 4: OAuth Configuration

**Question**: You're setting up OAuth for BigQuery in dbt Cloud. Which component needs to be configured in BOTH Google Cloud Console and dbt Cloud?

A) Service account key  
B) Client ID and Client Secret  
C) Project ID  
D) Dataset name

**Answer**: B) Client ID and Client Secret

**Explanation**:

- OAuth requires a Client ID and Client Secret generated in Google Cloud Console
- These values must then be entered in dbt Cloud project settings
- Service account keys are not used with OAuth
- Project ID and dataset name are connection parameters, not OAuth configuration

### Question 5: Security Best Practices

**Question**: Which of the following represents the BEST security practice for dbt Cloud warehouse connections?

A) Use ACCOUNTADMIN role to ensure dbt has all necessary permissions  
B) Create a dedicated role with minimal required permissions and use service accounts for production  
C) Share one service account across all environments to simplify management  
D) Store connection credentials in the dbt project repository for easy access

**Answer**: B) Create a dedicated role with minimal required permissions and use service accounts for production

**Explanation**:

- Follows the principle of least privilege
- Service accounts provide consistent, auditable access
- ACCOUNTADMIN grants excessive permissions (option A)
- Sharing credentials across environments (option C) violates separation of concerns
- Storing credentials in repositories (option D) is a critical security vulnerability

### Question 6: Troubleshooting Scenario

**Question**: A developer reports that they can successfully run dbt commands in the IDE, but when they commit code, the CI job fails with "Database connection failed." What should you check first?

A) Whether the developer has the correct permissions  
B) Whether the deployment environment has valid credentials configured  
C) Whether the Git repository is properly connected  
D) Whether the dbt project has syntax errors

**Answer**: B) Whether the deployment environment has valid credentials configured

**Explanation**:

- IDE uses development credentials (which work per the scenario)
- CI jobs use deployment environment credentials
- Since the issue only occurs in CI, it's environment-specific
- Git connection and syntax errors would present different error messages

## Summary

Configuring data warehouse connections properly is critical for dbt Cloud success. Key takeaways:

1. **Choose the right authentication method** for each use case:
   - OAuth for development
   - Key pairs for production
   - Service accounts for automation

2. **Implement proper security controls**:
   - IP allowlisting
   - Least privilege access
   - Credential separation by environment

3. **Test thoroughly** before deploying to production

4. **Maintain credentials** through regular rotation and monitoring

5. **Document everything** for team knowledge sharing and compliance

## Next Steps

Continue to [Topic 2: Configuring dbt Git Connections](./02-git-connections.md) to learn about version control integration.
