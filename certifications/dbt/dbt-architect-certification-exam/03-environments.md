# Topic 3: Creating and Maintaining dbt Environments

## Overview

Environments in dbt Cloud represent different stages of your analytics workflow (development, staging, production). Proper environment configuration ensures code is developed safely, tested thoroughly, and deployed reliably to production.

## Key Concepts and Definitions

### What is an Environment?

An **environment** in dbt Cloud is a configuration that defines:

- Which Git branch to use
- Which data warehouse credentials to use
- What schema/dataset to write to
- Which version of dbt Core to run
- Environment-specific variables and settings

### Environment Types

#### Development Environment

- Used by developers in the IDE
- Typically writes to personal or shared dev schemas
- Uses individual developer credentials (OAuth)
- Connected to feature branches
- Fast iteration and testing

#### Deployment Environment

- Used for production and staging jobs
- Writes to production or staging schemas
- Uses service account credentials
- Connected to stable branches (main, production)
- Scheduled and automated execution

### Key Environment Components

#### 1. Credentials

Authentication information for warehouse access:

- Development: Individual user OAuth or personal credentials
- Deployment: Service accounts with key pair authentication

#### 2. Schema/Dataset Configuration

Target location for dbt models:

- Custom schemas per environment
- Schema prefix/suffix configuration
- Dataset organization in BigQuery

#### 3. Environment Variables

Dynamic configuration values:

- Database names
- Schema overrides
- Feature flags
- API endpoints

#### 4. dbt Version

The dbt Core version to use:

- Latest stable
- Specific version pinning
- Version compatibility management

## Practical Examples

### Example 1: Creating a Development Environment

```markdown
Navigate to: Account Settings → Projects → [Your Project] → Environments

Click "Create Environment"

Configuration:
  Environment Name: Development
  Environment Type: Development
  dbt Version: 1.7 (latest)
  Default to Custom Branch: ✓ Enabled
  
Deployment Credentials: [None for dev environment]
```

**Development Schema Configuration:**

```yaml
# dbt_project.yml
models:
  my_project:
    +schema: "dbt_{{ target.name }}"
    
# Result in dev: DBT_DEV
# Result in prod: DBT_PROD
```

### Example 2: Creating a Production Deployment Environment

```markdown
Navigate to: Environments → Create Environment

General Settings:
  Name: Production
  Type: Deployment
  
Git Configuration:
  Custom Branch: main
  
Warehouse Connection:
  Dataset/Schema: ANALYTICS_PROD
  
Credentials:
  Auth Method: Key Pair
  Username: DBT_CLOUD_PROD_SERVICE
  Private Key: [Upload key file]
  
dbt Settings:
  dbt Version: 1.7 (versionless)
  Target Name: prod
  Threads: 8
  
Environment Variables:
  DBT_DATABASE: ANALYTICS
  DBT_PROD_SCHEMA: PROD
```

### Example 3: Configuring Environment Variables

#### In dbt Cloud UI

```markdown
Environment Settings → Environment Variables

Add Variables:
  DBT_SNOWFLAKE_DATABASE = ANALYTICS_PROD
  DBT_TARGET_SCHEMA = PRODUCTION
  ENABLE_FEATURE_X = true
  API_ENDPOINT = https://api.production.company.com
```

#### Using in dbt Project

```sql
-- models/example_model.sql
select
  {{ env_var('API_ENDPOINT') }} as api_endpoint,
  current_timestamp as loaded_at
from {{ source('app', 'events') }}
where environment = '{{ env_var('DBT_TARGET_SCHEMA') }}'
```

```yaml
# dbt_project.yml
models:
  my_project:
    materialized: view
    enabled: "{{ env_var('ENABLE_FEATURE_X', 'false') | as_bool }}"
```

### Example 4: Setting Up Environment Deferral

**Production Environment Setup:**

```markdown
Environment: Production
  Name: production
  Type: Deployment
  Custom Branch: main
```

**Staging Environment with Deferral:**

```markdown
Environment: Staging
  Name: staging
  Type: Deployment
  Custom Branch: develop
  
Deferral Configuration:
  ✓ Defer to another environment
  Defer to: production
```

**How Deferral Works:**

```bash
# When running in staging:
dbt run --select +my_new_model

# dbt will:
# 1. Build upstream models that exist in production (use production artifacts)
# 2. Only build my_new_model and models that changed since production
```

### Example 5: Rotating Key Pair Authentication via API

```python
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

private_pem = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.NoEncryption()
)

public_key = private_key.public_key()
public_pem = public_key.public_bytes(
    encoding=serialization.Encoding.OpenSSH,
    format=serialization.PublicFormat.OpenSSH
)

# Update Snowflake user with new public key
snowflake_conn.cursor().execute(f"""
    ALTER USER DBT_CLOUD_PROD_SERVICE
    SET RSA_PUBLIC_KEY = '{public_pem.decode("utf-8")}';
""")

# Update dbt Cloud environment credentials via API
response = requests.post(
    f'https://cloud.getdbt.com/api/v3/accounts/{account_id}/environments/{env_id}/credentials/',
    headers={
        'Authorization': f'Token {api_token}',
        'Content-Type': 'application/json'
    },
    json={
        'private_key': private_pem.decode('utf-8')
    }
)

print(f"Key rotation status: {response.status_code}")
```

### Example 6: Custom Schema Configuration

```yaml
# dbt_project.yml

# Strategy 1: Environment-based schemas
models:
  my_project:
    staging:
      +schema: staging_{{ target.name }}
      # dev: staging_dev
      # prod: staging_prod
    
    marts:
      +schema: marts_{{ target.name }}
      # dev: marts_dev
      # prod: marts_prod

# Strategy 2: Using environment variables
models:
  my_project:
    +schema: "{{ env_var('DBT_TARGET_SCHEMA', target.schema) }}"

# Strategy 3: Custom macro
models:
  my_project:
    +schema: "{{ generate_schema_name(schema) }}"
```

```sql
-- macros/generate_schema_name.sql
{% macro generate_schema_name(custom_schema_name, node) -%}
    {%- set default_schema = target.schema -%}
    
    {%- if target.name == 'prod' -%}
        {%- if custom_schema_name is none -%}
            {{ default_schema }}
        {%- else -%}
            {{ custom_schema_name | trim }}
        {%- endif -%}
    
    {%- else -%}
        {%- if custom_schema_name is none -%}
            {{ default_schema }}
        {%- else -%}
            {{ default_schema }}_{{ custom_schema_name | trim }}
        {%- endif -%}
    {%- endif -%}

{%- endmacro %}
```

## Best Practices

### 1. Separate Development and Production Credentials

**Development:**
```markdown
Credentials: Individual OAuth accounts
Purpose: Personal development and testing
Schema: DEV_[username] or shared DEV
Permissions: Read all, write to dev schemas only
```

**Production:**
```markdown
Credentials: Service account with key pair
Purpose: Automated production runs
Schema: PROD or ANALYTICS
Permissions: Read all, write to prod schemas only
```

### 2. Use Deferral to Optimize Build Times

```markdown
Scenario: Testing a new model in staging

Without Deferral:
  - Build all 500 upstream models (30 minutes)
  - Build new model (1 minute)
  Total: 31 minutes

With Deferral to Production:
  - Defer to 500 existing production models (0 minutes)
  - Build only new model (1 minute)
  Total: 1 minute
```

### 3. Implement Consistent Naming Conventions

```markdown
Environment Names:
  ✓ development, staging, production
  ✓ dev, stg, prod
  ✗ John's Environment, Test123, Prod-Final-v2

Schema Names:
  ✓ dbt_dev, dbt_staging, dbt_prod
  ✓ analytics_dev, analytics_prod
  ✗ temporary_schema, schema1, new_schema
```

### 4. Version Control for Environment Configurations

```yaml
# environments.yml (documentation, not config file)
environments:
  development:
    type: development
    dbt_version: 1.7
    custom_branch_enabled: true
    
  staging:
    type: deployment
    branch: develop
    dbt_version: 1.7
    target: stg
    threads: 4
    schema: ANALYTICS_STAGING
    defer_to: production
    
  production:
    type: deployment
    branch: main
    dbt_version: 1.7
    target: prod
    threads: 8
    schema: ANALYTICS_PROD
```

### 5. Regular Key Rotation Schedule

```markdown
Key Rotation Cadence:
  - Production credentials: Every 90 days
  - Staging credentials: Every 90 days
  - Development credentials: As needed

Rotation Process:
  1. Generate new key pair
  2. Add new public key to warehouse user
  3. Test connection with new key
  4. Update dbt Cloud environment
  5. Verify jobs run successfully
  6. Remove old public key after 24 hours
  7. Document rotation date
```

### 6. Use Environment Variables for Configuration

```yaml
# Good: Using environment variables
models:
  my_project:
    +database: "{{ env_var('DBT_DATABASE') }}"
    +schema: "{{ env_var('DBT_SCHEMA') }}"

# Bad: Hardcoding values
models:
  my_project:
    +database: PRODUCTION_DB  # Hard to change per environment
    +schema: ANALYTICS        # Same value everywhere
```

### 7. Configure Appropriate Thread Counts

```markdown
Thread Count Guidelines:

Development Environment:
  Threads: 4
  Reason: Multiple developers sharing resources

CI/Staging Environment:
  Threads: 4-8
  Reason: Fast feedback, moderate resource usage

Production Environment:
  Threads: 8-16
  Reason: Maximum performance for scheduled runs
  
Considerations:
  - Warehouse size/concurrency limits
  - Model dependency structure
  - Cost implications
  - Other concurrent workloads
```

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Share Production Credentials Across Environments

```markdown
# BAD: Same credentials everywhere
Development Environment: PROD_SERVICE_ACCOUNT
Staging Environment: PROD_SERVICE_ACCOUNT
Production Environment: PROD_SERVICE_ACCOUNT
```

**Why it's bad:**

- Developers could accidentally modify production data
- No isolation between environments
- Difficult to audit who changed what
- Single point of failure

**Solution:** Use separate credentials per environment with appropriate permissions

### ❌ DON'T Use Personal Credentials for Production Jobs

```markdown
# BAD: Production job using personal account
Production Environment:
  Username: john.doe@company.com
  Password: John's password
```

**Why it's bad:**

- Jobs break when employee leaves
- No consistent identity for automation
- Difficult to audit
- Password resets affect production

**Solution:** Use dedicated service accounts

### ❌ DON'T Hardcode Environment-Specific Values

```sql
-- BAD: Hardcoded database names
select * from PRODUCTION_DB.ANALYTICS.customers

-- GOOD: Using target or env_var
select * from {{ target.database }}.{{ target.schema }}.customers

-- BETTER: Using sources
select * from {{ source('erp', 'customers') }}
```

### ❌ DON'T Skip Testing in Non-Production Environments

```markdown
# BAD: Only production environment configured
Environments:
  - Production (main branch)

Workflow:
  Developer → Commit to main → Production run → Hope it works
```

**Why it's bad:**

- No testing before production
- Errors affect production data
- No rollback strategy
- Expensive mistakes

**Solution:** Always have dev/staging environments for testing

### ❌ DON'T Ignore dbt Version Compatibility

```markdown
# BAD: Using different major versions
Development: dbt 1.7
Production: dbt 1.4

Result: Code works in dev but breaks in prod due to version differences
```

**Why it's bad:**

- Syntax differences between versions
- Feature availability varies
- Unexpected behavior in production

**Solution:** Keep versions consistent or test upgrades thoroughly

## Real-World Scenarios and Solutions

### Scenario 1: Multi-Environment Setup for Enterprise

**Challenge:** Set up a complete environment structure for a large analytics team with proper isolation and governance.

**Solution:**

```markdown
Environment Structure:

1. Development (Personal Sandboxes)
   Type: Development
   Branch: Any (custom branch enabled)
   Schema: DBT_{{ user.name }}
   Credentials: Individual OAuth
   Purpose: Individual developer work
   
2. Integration (Shared Dev)
   Type: Deployment
   Branch: develop
   Schema: DBT_INTEGRATION
   Credentials: Integration service account
   Purpose: Integration testing before PR
   Defer to: Production
   
3. Staging (Pre-Production)
   Type: Deployment
   Branch: staging
   Schema: DBT_STAGING
   Credentials: Staging service account
   Purpose: Final validation before production
   Defer to: Production
   
4. Production
   Type: Deployment
   Branch: main
   Schema: ANALYTICS
   Credentials: Production service account
   Purpose: Production data transformations
```

**Workflow:**

```markdown
1. Developer creates feature branch from develop
2. Develops in personal development environment
3. Commits and creates PR to develop
4. CI job runs in Integration environment (deferred)
5. After review, merge to develop
6. Integration job runs full build
7. When stable, create PR to staging branch
8. Staging job runs (deferred to production)
9. After stakeholder approval, merge to main
10. Production job runs on schedule
```

### Scenario 2: Environment Variable Management Across Environments

**Challenge:** Different API endpoints, feature flags, and configurations needed per environment.

**Solution:**

```markdown
Development Environment Variables:
  DBT_DATABASE = ANALYTICS_DEV
  DBT_SCHEMA = DEV
  API_ENDPOINT = https://api.dev.company.com
  ENABLE_EXPERIMENTAL_FEATURES = true
  DEBUG_MODE = true
  DATA_RETENTION_DAYS = 7

Staging Environment Variables:
  DBT_DATABASE = ANALYTICS_STAGING
  DBT_SCHEMA = STAGING
  API_ENDPOINT = https://api.staging.company.com
  ENABLE_EXPERIMENTAL_FEATURES = true
  DEBUG_MODE = false
  DATA_RETENTION_DAYS = 30

Production Environment Variables:
  DBT_DATABASE = ANALYTICS
  DBT_SCHEMA = PROD
  API_ENDPOINT = https://api.company.com
  ENABLE_EXPERIMENTAL_FEATURES = false
  DEBUG_MODE = false
  DATA_RETENTION_DAYS = 365
  COMPLIANCE_MODE = true
```

**Usage in Models:**

```sql
-- models/events_filtered.sql
select
  event_id,
  event_type,
  user_id,
  occurred_at
from {{ source('raw', 'events') }}
where occurred_at >= current_date - interval '{{ env_var("DATA_RETENTION_DAYS") }}' day
{% if env_var('DEBUG_MODE') == 'true' %}
  and user_id in (select user_id from {{ ref('test_users') }})
{% endif %}
```

### Scenario 3: Automatic Key Rotation System

**Challenge:** Implement automated key rotation for all deployment environments to maintain security compliance.

**Solution:**

```python
# key_rotation_automation.py
import os
from datetime import datetime, timedelta
from typing import Dict, List
import requests
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend

class KeyRotationManager:
    def __init__(self, account_id: str, api_token: str):
        self.account_id = account_id
        self.api_token = api_token
        self.base_url = "https://cloud.getdbt.com/api/v3"
        
    def generate_key_pair(self) -> Dict[str, bytes]:
        """Generate new RSA key pair"""
        private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=2048,
            backend=default_backend()
        )
        
        private_pem = private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=serialization.NoEncryption()
        )
        
        public_key = private_key.public_key()
        public_pem = public_key.public_bytes(
            encoding=serialization.Encoding.OpenSSH,
            format=serialization.PublicFormat.OpenSSH
        )
        
        return {
            'private': private_pem,
            'public': public_pem
        }
    
    def rotate_environment_key(self, environment_id: str, 
                              snowflake_user: str) -> bool:
        """Rotate key for a single environment"""
        # Generate new key pair
        keys = self.generate_key_pair()
        
        # Update Snowflake user
        self.update_snowflake_key(snowflake_user, keys['public'])
        
        # Update dbt Cloud credentials
        self.update_dbt_credentials(environment_id, keys['private'])
        
        # Test the connection
        return self.test_connection(environment_id)
    
    def update_snowflake_key(self, user: str, public_key: bytes):
        """Update Snowflake user's public key"""
        # Implementation depends on your Snowflake connection method
        pass
    
    def update_dbt_credentials(self, environment_id: str, 
                              private_key: bytes):
        """Update dbt Cloud environment credentials"""
        response = requests.post(
            f'{self.base_url}/accounts/{self.account_id}/environments/{environment_id}/credentials/',
            headers={
                'Authorization': f'Token {self.api_token}',
                'Content-Type': 'application/json'
            },
            json={
                'private_key': private_key.decode('utf-8')
            }
        )
        response.raise_for_status()
    
    def test_connection(self, environment_id: str) -> bool:
        """Test environment connection"""
        response = requests.post(
            f'{self.base_url}/accounts/{self.account_id}/environments/{environment_id}/test/',
            headers={'Authorization': f'Token {self.api_token}'}
        )
        return response.status_code == 200

# Usage
manager = KeyRotationManager(
    account_id='12345',
    api_token=os.environ['DBT_CLOUD_API_TOKEN']
)

environments = [
    {'id': '101', 'name': 'staging', 'snowflake_user': 'DBT_STAGING'},
    {'id': '102', 'name': 'production', 'snowflake_user': 'DBT_PROD'}
]

for env in environments:
    print(f"Rotating key for {env['name']}...")
    success = manager.rotate_environment_key(env['id'], env['snowflake_user'])
    print(f"  {'✓ Success' if success else '✗ Failed'}")
```

### Scenario 4: Migrating Between Environments

**Challenge:** You need to promote code from staging to production, but the environments have different configurations.

**Solution:**

```markdown
Pre-Migration Checklist:
  ☐ All tests passing in staging
  ☐ Code reviewed and approved
  ☐ Environment variables documented
  ☐ Schema names verified
  ☐ Service account permissions confirmed
  ☐ Downstream dependencies identified
  ☐ Rollback plan documented

Migration Steps:

1. Verify Staging Success
   - Review staging job logs
   - Confirm data quality tests pass
   - Validate with business stakeholders

2. Update Production Environment Variables (if needed)
   Example differences:
     Staging: ENABLE_NEW_LOGIC = true
     Production: ENABLE_NEW_LOGIC = false (until cutover)

3. Create Production Release PR
   - Merge staging → main
   - Trigger production CI job
   - Review manifest.json differences

4. Schedule Production Deployment
   - Choose low-traffic window
   - Notify stakeholders
   - Enable monitoring alerts

5. Execute Deployment
   - Trigger production job manually (first time)
   - Monitor execution
   - Validate results

6. Post-Deployment Validation
   - Run data quality checks
   - Compare row counts with previous run
   - Check downstream systems
   - Monitor for errors

7. Enable Scheduled Runs
   - If validation successful, enable schedule
   - Document deployment in runbook
```

## Sample Exam Questions

### Question 1: Environment Types

**Question:** What is the PRIMARY difference between a Development environment and a Deployment environment in dbt Cloud?

A) Development environments can only use OAuth authentication  
B) Deployment environments are used to run jobs, while Development environments are for IDE usage  
C) Development environments cannot access production data  
D) Deployment environments require service account credentials

**Answer:** B) Deployment environments are used to run jobs, while Development environments are for IDE usage

**Explanation:**

- Development environments are specifically designed for IDE usage
- Deployment environments are designed for scheduled and triggered jobs
- Both can use various authentication methods (though best practices differ)
- Both can access production data (though permissions should differ)

### Question 2: Deferral Configuration

**Question:** Your staging environment is configured to defer to production. When you run `dbt run --select +my_new_model` in staging, what happens?

A) All models including upstream dependencies are built in staging  
B) Only my_new_model is built, upstream models cause errors  
C) Upstream models reference production, only my_new_model and modified upstream models are built in staging  
D) The command fails because deferral doesn't work with selection syntax

**Answer:** C) Upstream models reference production, only my_new_model and modified upstream models are built in staging

**Explanation:**

- Deferral allows staging to reference existing production models
- Only new or modified models are built in staging
- This significantly reduces build time and resource usage
- Selection syntax works normally with deferral

### Question 3: Environment Variables

**Question:** You have an environment variable `DBT_DATABASE` set to `ANALYTICS_PROD` in your production environment. How would you reference this in a model?

A) `{{ var('DBT_DATABASE') }}`  
B) `{{ env_var('DBT_DATABASE') }}`  
C) `$DBT_DATABASE`  
D) `{{ target.DBT_DATABASE }}`

**Answer:** B) `{{ env_var('DBT_DATABASE') }}`

**Explanation:**

- `env_var()` is the Jinja function for accessing environment variables
- `var()` is for dbt project variables, not environment variables
- `$VARIABLE` is shell syntax, not dbt Jinja syntax
- `target` object contains connection info but not custom environment variables

### Question 4: Key Rotation

**Question:** Your security team requires rotating service account keys every 90 days. What is the BEST approach to rotate keys for a dbt Cloud deployment environment?

A) Delete and recreate the environment with new keys  
B) Use the dbt Cloud API to update the private key while keeping the environment configuration  
C) Contact dbt Support to rotate keys manually  
D) Key rotation is not supported for deployment environments

**Answer:** B) Use the dbt Cloud API to update the private key while keeping the environment configuration

**Explanation:**

- The API supports credential updates without environment recreation
- This maintains job configurations, schedules, and settings
- Environment recreation would lose configuration and history
- Key rotation is supported and documented

### Question 5: Schema Configuration

**Question:** You want developers to write to separate schemas based on their username, while production writes to a shared ANALYTICS schema. Which configuration achieves this?

A) Use `+schema: "{{ target.name }}"` in dbt_project.yml  
B) Use custom `generate_schema_name()` macro that checks `target.name`  
C) Set different schema names in each environment's configuration  
D) Use environment variables for schema names

**Answer:** B) Use custom `generate_schema_name()` macro that checks `target.name`

**Explanation:**

- The `generate_schema_name()` macro provides the most flexibility
- It can implement different logic for dev vs. prod targets
- Option A would create schemas based on target name, not username
- Option C sets schema at environment level but doesn't support dynamic per-user schemas
- Option D could work but is less elegant than the built-in macro approach

### Question 6: Access Control

**Question:** A developer reports they cannot run models in the staging environment, receiving "Permission denied" errors. They can successfully run models in the development environment. What is the MOST likely cause?

A) The staging environment's service account doesn't have write permissions to the staging schema  
B) The developer doesn't have permission to access the staging environment in dbt Cloud  
C) The Git branch is not properly configured  
D) The dbt version in staging is incompatible with their code

**Answer:** A) The staging environment's service account doesn't have write permissions to the staging schema

**Explanation:**

- Deployment environments (like staging) use service account credentials
- Development environments use individual credentials
- If dev works but staging doesn't, it's likely a credential/permission difference
- dbt Cloud access (B) would prevent seeing the environment, not running in it
- Git and version issues (C, D) would show different error messages

## Summary

Proper environment configuration is essential for safe, efficient dbt development and deployment. Key takeaways:

1. **Separate environments by purpose:**
   - Development: Individual developer work
   - Deployment: Automated scheduled runs

2. **Use appropriate credentials:**
   - Development: Individual OAuth accounts
   - Production: Service accounts with key pairs

3. **Leverage deferral** to reduce build times and resource usage

4. **Manage environment variables** for configuration flexibility

5. **Implement regular key rotation** for security compliance

6. **Configure schemas appropriately** for environment isolation

7. **Test thoroughly** in non-production environments before deploying

## Next Steps

Continue to [Topic 4: Creating and Maintaining Job Definitions](./04-job-definitions.md) to learn about job configuration and scheduling.
