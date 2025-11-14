# Topic 5: Configuring dbt Security and Licenses

## Overview

Security and license management in dbt Cloud ensures proper access control, compliance, and efficient resource allocation across your organization. This topic covers authentication, authorization, license management, and security best practices.

## Key Concepts and Definitions

### Authentication vs Authorization

- **Authentication**: Verifying who you are (login credentials, SSO)
- **Authorization**: Determining what you can do (permissions, roles)

### Permission Sets

Pre-defined collections of permissions in dbt Cloud:

- **Account Admin**: Full account-level access
- **Admin**: Project-level administrative access
- **Developer**: Can develop in IDE, view jobs
- **Analyst**: Read-only access to documentation and artifacts
- **Job Admin**: Can create and manage jobs
- **Read-Only**: View-only access

### RBAC (Role-Based Access Control)

Enterprise feature allowing custom role definitions with granular permissions.

### Service Tokens

API tokens for programmatic access:

- **User tokens**: Associated with individual users
- **Service account tokens**: For automation and integrations

### License Types

- **Developer License**: Full IDE and development access
- **Read-Only License**: Documentation and artifact viewing
- **IT License**: Administrative access without consuming developer seats

## Practical Examples

### Example 1: Creating Service Tokens

**User Token Creation:**

```markdown
Navigate to: Profile → API Access

Create Token:
  Token Name: Personal API Access
  Permissions: Inherited from user permissions
  Expiration: 90 days (or per policy)
  
Usage:
  curl -H "Authorization: Token dbt_abc123..." \
    https://cloud.getdbt.com/api/v2/accounts/12345/projects/
```

**Service Account Token Creation (Enterprise):**

```markdown
Navigate to: Account Settings → Service Tokens

Create Service Token:
  Token Name: CI/CD Integration
  Permissions: 
    - Read all projects
    - Trigger jobs
    - Read run artifacts
  Token Type: Service Account
  
Usage: CI/CD pipelines, external orchestration
```

### Example 2: Assigning Permission Sets

```markdown
Navigate to: Account Settings → Team → [User]

Assign Permissions:

Project: Analytics Platform
  Permission Set: Developer
  Access: Full IDE access, can run jobs manually
  
Project: Finance Models
  Permission Set: Read-Only
  Access: View documentation only
  
Account Level:
  Permission Set: Member
  Access: Can see account, no admin privileges
```

### Example 3: Creating License Mappings

```markdown
Navigate to: Account Settings → Licenses

License Pool Configuration:
  Developer Licenses: 25 assigned / 30 total
  Read-Only Licenses: 50 assigned / 100 total
  
Assign License to User:
  User: jane.doe@company.com
  License Type: Developer
  Auto-assign to groups: Data Team
  
License Groups:
  Group: Data Engineering
    License Type: Developer
    Members: Auto-assigned based on SSO group
    
  Group: Business Analysts
    License Type: Read-Only
    Members: Auto-assigned based on SSO group
```

### Example 4: Configuring SSO (SAML)

**For Okta:**

```markdown
Step 1: Configure in Okta
  Application: dbt Cloud
  Sign-on method: SAML 2.0
  
  ACS URL: 
    https://cloud.getdbt.com/complete/saml
    
  Entity ID:
    https://cloud.getdbt.com
    
  Attribute Statements:
    - email → user.email
    - firstName → user.firstName
    - lastName → user.lastName
    
  Group Attribute Statements:
    - groups → matches regex .* (all groups)

Step 2: Configure in dbt Cloud
  Navigate to: Account Settings → Single Sign-On
  
  Identity Provider: Custom SAML
  SSO URL: [From Okta]
  Entity ID: [From Okta]
  X.509 Certificate: [From Okta]
  
  Group Mapping:
    Okta Group: data-engineering
    dbt Permission Set: Developer
    
    Okta Group: business-users
    dbt Permission Set: Read-Only

Step 3: Test SSO
  - Test login with SSO user
  - Verify group memberships
  - Confirm permissions applied correctly
```

### Example 5: Implementing RBAC (Enterprise)

```markdown
Navigate to: Account Settings → Access Control → Roles

Create Custom Role: Data Pipeline Engineer

Permissions:
  Projects:
    ✓ View project
    ✓ Edit project settings
    ✓ View environments
    ✓ Edit environments
    
  Jobs:
    ✓ View jobs
    ✓ Create jobs
    ✓ Edit jobs
    ✓ Run jobs
    ✓ View run history
    
  IDE:
    ✓ Use IDE
    ✓ Commit code
    
  Credentials:
    ✓ Manage development credentials
    ✗ Manage deployment credentials (restricted)
    
  Documentation:
    ✓ View documentation
    ✓ Generate documentation

Assign Role:
  Users: Data engineering team
  Scope: All analytics projects
```

### Example 6: Managing Users via API

```python
import requests

API_TOKEN = "your_token"
ACCOUNT_ID = "12345"
BASE_URL = "https://cloud.getdbt.com/api/v2"

headers = {
    "Authorization": f"Token {API_TOKEN}",
    "Content-Type": "application/json"
}

# List all users
response = requests.get(
    f"{BASE_URL}/accounts/{ACCOUNT_ID}/users/",
    headers=headers
)
users = response.json()["data"]

# Add new user
new_user = {
    "email": "new.developer@company.com",
    "license_type": "developer",
    "groups": ["data-engineering"]
}

response = requests.post(
    f"{BASE_URL}/accounts/{ACCOUNT_ID}/users/",
    headers=headers,
    json=new_user
)

# Remove user
user_id = 67890
response = requests.delete(
    f"{BASE_URL}/accounts/{ACCOUNT_ID}/users/{user_id}/",
    headers=headers
)

# Bulk license assignment
for user in users:
    if user["email"].endswith("@contractor.com"):
        # Assign read-only license to contractors
        requests.patch(
            f"{BASE_URL}/accounts/{ACCOUNT_ID}/users/{user['id']}/",
            headers=headers,
            json={"license_type": "read_only"}
        )
```

## Best Practices

### 1. Principle of Least Privilege

```markdown
✓ Grant minimum permissions needed for job function
✓ Use read-only licenses for view-only users
✓ Restrict production access to authorized personnel
✓ Regular access reviews (quarterly)

Example Access Matrix:

Role: Junior Analyst
  - Read-Only license
  - Can view documentation
  - Cannot access IDE or modify code

Role: Analytics Engineer
  - Developer license
  - Full IDE access to analytics projects
  - Cannot modify infrastructure or security settings

Role: Data Platform Admin
  - IT license
  - Account admin permissions
  - Can manage all settings and users
```

### 2. Implement SSO for Enterprise

```markdown
Benefits:
  ✓ Centralized user management
  ✓ Automatic onboarding/offboarding
  ✓ Stronger authentication (MFA)
  ✓ Group-based access control
  ✓ Audit trail through IdP
  
Required for organizations with:
  - 50+ users
  - Compliance requirements (SOC 2, HIPAA)
  - Multiple teams/departments
  - High security requirements
```

### 3. Regular Token Rotation

```markdown
Service Token Rotation Schedule:
  - Production tokens: Every 90 days
  - CI/CD tokens: Every 90 days
  - Personal access tokens: Every 180 days
  - Emergency/temporary tokens: After use or 7 days max

Rotation Process:
  1. Generate new token
  2. Update systems using old token
  3. Test new token
  4. Revoke old token after 24-hour grace period
  5. Document rotation date
```

### 4. License Management Strategy

```markdown
Optimize License Allocation:

1. Audit Current Usage:
   - Who has licenses?
   - Who actively uses them?
   - Are there inactive users?

2. Categorize Users:
   Developer Licenses:
     - Active code contributors
     - Data engineers
     - Analytics engineers
   
   Read-Only Licenses:
     - Business analysts (view docs only)
     - Stakeholders
     - Auditors
   
   No License Needed:
     - Users who only receive reports
     - External partners (use sharing features)

3. Implement License Pool:
   - Shared pool for temporary access
   - Contractor short-term licenses
   - Training/onboarding licenses
```

### 5. Audit Trail Maintenance

```markdown
What to Log:
  ✓ User login/logout events
  ✓ Permission changes
  ✓ License assignments
  ✓ Job executions
  ✓ Code deployments
  ✓ API token usage
  ✓ Failed authentication attempts

Retention Policy:
  - Active logs: 90 days in dbt Cloud
  - Archived logs: 7 years (export to SIEM)
  - Compliance logs: Per regulatory requirements

Review Schedule:
  - Weekly: Failed authentication attempts
  - Monthly: Permission changes
  - Quarterly: Full access audit
```

### 6. Secure Service Token Storage

```markdown
❌ Never store tokens in:
  - Git repositories
  - Plain text files
  - Application code
  - Unencrypted databases
  - Shared documents

✓ Store tokens in:
  - Secret managers (AWS Secrets Manager, Azure Key Vault)
  - CI/CD platform secrets (GitHub Secrets, GitLab CI Variables)
  - Environment variables (encrypted)
  - Password managers (for personal tokens)

Example: GitHub Actions

name: dbt Run
on: [push]
jobs:
  dbt:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run dbt
        env:
          DBT_CLOUD_TOKEN: ${{ secrets.DBT_CLOUD_TOKEN }}
        run: |
          curl -X POST \
            -H "Authorization: Token $DBT_CLOUD_TOKEN" \
            https://cloud.getdbt.com/api/v2/accounts/...
```

## Anti-Patterns (What NOT to Do)

### ❌ DON'T Share Accounts or Tokens

```markdown
# BAD: Shared service account
Username: team@company.com
Password: [shared with entire team]
API Token: [used by multiple people]

Problems:
  - No individual accountability
  - Can't revoke access for one person
  - Audit trail shows "team", not individuals
  - Security risk if one person leaves
```

### ❌ DON'T Give Everyone Admin Access

```markdown
# BAD: Everyone is admin
All 50 users: Account Admin permission

Risks:
  - Anyone can delete projects
  - Anyone can modify security settings
  - No separation of duties
  - Compliance violations
```

### ❌ DON'T Ignore Inactive Users

```markdown
# BAD: Never remove old users
User: john.doe@company.com
Last Login: 2 years ago
Status: Active with Developer license
Access: Full permissions to production

Problems:
  - Wasted license
  - Security risk if account compromised
  - Compliance issues
  - Paying for unused seats
```

### ❌ DON'T Use Personal Accounts for Automation

```markdown
# BAD: CI/CD using personal account
CI Pipeline:
  dbt Cloud Token: john.doe's personal token
  
Result when John leaves:
  - Token revoked
  - All CI/CD pipelines break
  - Production deployments fail
  - Emergency to fix
```

### ❌ DON'T Skip MFA/SSO

```markdown
# BAD: Username/password only
Authentication: Basic password
MFA: Disabled
SSO: Not configured

Risks:
  - Weak passwords
  - Phishing attacks
  - Credential stuffing
  - No centralized control
```

## Real-World Scenarios and Solutions

### Scenario 1: Onboarding New Team Members at Scale

**Challenge:** Onboard 20 new data analysts efficiently with appropriate access.

**Solution:**

```markdown
Step 1: Create SSO Group Mappings
  IdP Group: data-analysts-new-hire
  dbt Permission: Read-Only
  Auto-assign License: Read-Only
  Projects: All (view only)

Step 2: Onboarding Process
  1. HR adds to data-analysts-new-hire group in Okta
  2. User logs into dbt Cloud via SSO (auto-provisioned)
  3. Automatically assigned read-only license
  4. Can view documentation immediately
  
Step 3: Promotion to Developer (After training)
  1. Move to data-analysts-active group
  2. dbt permission changes to Developer
  3. License upgraded automatically
  4. Full IDE access granted

Benefits:
  - No manual user creation
  - Consistent permissions
  - Self-service access
  - Audit trail through IdP
```

### Scenario 2: Securing Production Access

**Challenge:** Ensure only authorized engineers can deploy to production while allowing broad development access.

**Solution:**

```markdown
Environment-Based Access Control:

Development Environment:
  - All developers have access
  - OAuth authentication (individual accounts)
  - Can run models in dev schemas
  
Staging Environment:
  - Senior developers only
  - Service account (via RBAC)
  - Can trigger staging jobs
  
Production Environment:
  - Data platform team only (5 people)
  - Separate service account
  - Two-person approval for changes
  - All changes logged and alerted

Implementation:
  1. Create custom RBAC role "Production Deployer"
  2. Assign only to authorized users
  3. Production environment uses separate credentials
  4. Job triggers restricted by role
  5. Audit log monitoring for production changes
```

### Scenario 3: Managing Contractor Access

**Challenge:** Provide temporary access to contractors working on a 3-month project.

**Solution:**

```markdown
Contractor Access Policy:

License Assignment:
  - Type: Developer (from contractor pool)
  - Duration: Project length + 1 week
  - Auto-expire: Enabled
  
Permissions:
  - Project: Contractor-specific project only
  - Permission Set: Developer
  - Restrictions:
    ✗ Cannot access production
    ✗ Cannot create new projects
    ✗ Cannot view other projects
    ✓ Can develop in assigned project
    ✓ Can commit to feature branches

Process:
  1. Contractor start date:
     - Create account via API
     - Assign temp developer license
     - Add to contractor project only
     - Set expiration date
     
  2. During engagement:
     - Monitor activity weekly
     - Review code contributions
     - Extend expiration if needed
     
  3. Project end:
     - Download contractor's work
     - Revoke all access
     - Archive account
     - Return license to pool

Automation Script:
```python
from datetime import datetime, timedelta

def provision_contractor(email, project_id, duration_days=90):
    expiration = datetime.now() + timedelta(days=duration_days)
    
    user = create_user(
        email=email,
        license_type="developer",
        expiration_date=expiration,
        groups=["contractors"]
    )
    
    assign_permissions(
        user_id=user["id"],
        project_id=project_id,
        permission_set="developer",
        restrictions=["no_production_access"]
    )
    
    # Schedule reminder 1 week before expiration
    schedule_reminder(
        date=expiration - timedelta(days=7),
        message=f"Contractor {email} access expires in 7 days"
    )
    
    return user
```

### Scenario 4: Implementing Compliance Controls

**Challenge:** Meet SOC 2 compliance requirements for access control and audit trails.

**Solution:**

```markdown
Compliance Requirements & Implementation:

1. Access Control
   Requirement: Principle of least privilege
   Implementation:
     - RBAC with custom roles
     - Regular access reviews (quarterly)
     - Approval workflow for privilege elevation
     - Documented access matrix

2. Authentication
   Requirement: Strong authentication, MFA
   Implementation:
     - SSO required (no username/password)
     - MFA enforced at IdP level
     - Session timeout: 8 hours
     - Failed login lockout: 5 attempts

3. Audit Trail
   Requirement: Complete activity logging
   Implementation:
     - All actions logged with user attribution
     - Logs exported to SIEM (Splunk/Datadog)
     - 7-year retention
     - Monthly audit report generation

4. Separation of Duties
   Requirement: No single person has complete control
   Implementation:
     - Developer can write code
     - Reviewer can approve PR
     - Platform admin can deploy (different person)
     - Audit team has read-only access

5. Access Reviews
   Schedule:
     - Quarterly: Full user access audit
     - Monthly: Admin permission review
     - Weekly: Production access review
   
   Automated Reporting:
```python
def generate_access_audit_report():
    report = {
        "total_users": count_users(),
        "admin_users": get_users_by_permission("admin"),
        "production_access": get_production_access_users(),
        "inactive_users": get_inactive_users(days=90),
        "license_utilization": calculate_license_usage(),
        "failed_logins": get_failed_login_attempts(days=30)
    }
    
    # Flag anomalies
    if report["inactive_users"] > 5:
        alert("Multiple inactive users with active access")
    
    if report["failed_logins"] > 100:
        alert("Unusual number of failed login attempts")
    
    return report
```

## Sample Exam Questions

### Question 1: Permission Sets

**Question:** What is the appropriate permission set for a business analyst who needs to view dbt documentation and lineage but should NOT have access to write or modify code?

A) Developer  
B) Admin  
C) Read-Only  
D) Job Admin

**Answer:** C) Read-Only

**Explanation:**

- Read-Only provides documentation access without code modification rights
- Developer would give unnecessary IDE access
- Admin would give too many permissions
- Job Admin is for managing jobs, not viewing documentation

### Question 2: Service Tokens

**Question:** Your CI/CD pipeline needs to trigger dbt Cloud jobs. What is the BEST authentication method?

A) Use a developer's personal API token  
B) Create a service account token with job trigger permissions  
C) Use SSO credentials  
D) Share the account admin token

**Answer:** B) Create a service account token with job trigger permissions

**Explanation:**

- Service tokens are designed for automation
- Personal tokens break when user leaves
- SSO is for interactive login, not API access
- Sharing admin tokens violates security principles

### Question 3: License Types

**Question:** Your organization has 10 active data engineers who write dbt code and 50 business analysts who only view documentation. What license allocation is most appropriate?

A) 60 Developer licenses  
B) 10 Developer licenses, 50 Read-Only licenses  
C) 60 Read-Only licenses  
D) 10 Developer licenses, no licenses for analysts

**Answer:** B) 10 Developer licenses, 50 Read-Only licenses

**Explanation:**

- Developers need Developer licenses for IDE access
- Analysts only viewing docs can use Read-Only licenses
- This optimizes license costs
- Option D would prevent analysts from accessing documentation

### Question 4: SSO Configuration

**Question:** When configuring SAML SSO with group mapping, what happens when a user is removed from the "data-engineering" group in your IdP?

A) The user is immediately locked out of dbt Cloud  
B) The user's dbt Cloud permissions are updated at next login  
C) Nothing changes; permissions must be updated manually  
D) The user's account is deleted

**Answer:** B) The user's dbt Cloud permissions are updated at next login

**Explanation:**

- SSO group mappings sync permissions at login
- Not immediate; requires re-authentication
- Permissions not manual when SSO is configured
- Accounts aren't deleted automatically

### Question 5: RBAC Implementation

**Question:** You need to create a role that allows users to run jobs and view results but NOT create or modify job definitions. Which permissions should be included?

A) View jobs, Run jobs, View run history  
B) View jobs, Edit jobs, Run jobs  
C) Admin  
D) View jobs only

**Answer:** A) View jobs, Run jobs, View run history

**Explanation:**

- View jobs allows seeing job definitions
- Run jobs allows executing them
- View run history allows checking results
- Edit jobs would allow modification (not wanted)
- Admin is too broad
- View jobs only wouldn't allow running them

### Question 6: Security Best Practice

**Question:** A developer is leaving the company. What is the CORRECT sequence of actions?

A) Revoke dbt Cloud access → Remove from IdP → Archive projects  
B) Remove from IdP → Verify dbt Cloud access revoked → Transfer ownership of artifacts  
C) Delete user from dbt Cloud → Delete from IdP  
D) Disable account temporarily for 30 days

**Answer:** B) Remove from IdP → Verify dbt Cloud access revoked → Transfer ownership of artifacts

**Explanation:**

- Start with IdP removal (central auth source)
- SSO integration automatically restricts dbt Cloud access
- Transfer ownership before complete removal
- Proper offboarding sequence maintains audit trail

## Summary

Proper security and license management protects your data and optimizes costs. Key takeaways:

1. **Implement least privilege**: Grant minimum necessary permissions

2. **Use appropriate license types**:
   - Developer for code writers
   - Read-Only for documentation viewers
   - IT for administrators

3. **Enable SSO** for centralized authentication and group management

4. **Rotate tokens regularly**: Service tokens every 90 days

5. **Leverage RBAC** (Enterprise) for granular permission control

6. **Audit access regularly**: Quarterly full reviews, remove inactive users

7. **Secure service tokens**: Use secret managers, never commit to Git

8. **Document access policies** and compliance controls

## Next Steps

Continue to [Topic 6: Setting up Monitoring and Alerting for Jobs](./06-monitoring-alerting.md) to learn about job monitoring and notification strategies.
