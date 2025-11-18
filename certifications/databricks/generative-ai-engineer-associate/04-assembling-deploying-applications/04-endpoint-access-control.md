# Controlling Access to Model Serving Endpoint Resources

## Overview

Implement access control for Databricks Model Serving endpoints using Unity Catalog permissions, token-based authentication, and network security.

## Access Control Layers

| Layer | Controls | Implementation |
|-------|----------|----------------|
| **Unity Catalog** | Model read/write permissions | GRANT/REVOKE statements |
| **Endpoint** | Query permissions | Databricks ACLs |
| **Network** | IP allowlists, private links | Workspace settings |
| **Authentication** | Token/OAuth | Personal access tokens, service principals |

## Hands-on Examples

### Unity Catalog Model Permissions

```sql
-- Grant model usage permissions
GRANT EXECUTE ON MODEL main.ml_models.rag_chatbot TO `data-science-team`;

-- Grant model management permissions
GRANT ALL PRIVILEGES ON MODEL main.ml_models.rag_chatbot TO `ml-admins`;

-- Revoke permissions
REVOKE EXECUTE ON MODEL main.ml_models.rag_chatbot FROM `external-contractors`;

-- View permissions
SHOW GRANTS ON MODEL main.ml_models.rag_chatbot;
```

### Endpoint Access Control (Python SDK)

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import EndpointPermission, EndpointPermissionLevel

w = WorkspaceClient()

endpoint_name = "rag-chatbot-endpoint"

# Grant query permission to group
w.serving_endpoints.set_permissions(
    serving_endpoint_id=endpoint_name,
    access_control_list=[
        EndpointPermission(
            group_name="data-scientists",
            permission_level=EndpointPermissionLevel.CAN_QUERY
        ),
        EndpointPermission(
            group_name="ml-engineers",
            permission_level=EndpointPermissionLevel.CAN_MANAGE
        )
    ]
)

# Get current permissions
permissions = w.serving_endpoints.get_permissions(endpoint_name)
for acl in permissions.access_control_list:
    print(f"{acl.group_name or acl.user_name}: {acl.all_permissions}")
```

### Service Principal for Endpoint Access

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.iam import ServicePrincipal

# Create service principal
w = WorkspaceClient()

sp = w.service_principals.create(
    display_name="rag-chatbot-app",
    active=True
)

print(f"Created service principal: {sp.application_id}")

# Grant endpoint access to service principal
w.serving_endpoints.update_permissions(
    serving_endpoint_id="rag-chatbot-endpoint",
    access_control_list=[
        EndpointPermission(
            service_principal_name=sp.application_id,
            permission_level=EndpointPermissionLevel.CAN_QUERY
        )
    ]
)

# Generate token for service principal (done via UI or API)
# Use application_id and secret for authentication
```

### Token-Based Authentication

```python
import requests
import os

# Using personal access token
def query_endpoint_with_pat(endpoint_name: str, query: str):
    """Query endpoint using personal access token."""
    
    workspace_url = os.getenv("DATABRICKS_HOST")
    token = os.getenv("DATABRICKS_TOKEN")
    
    endpoint_url = f"{workspace_url}/serving-endpoints/{endpoint_name}/invocations"
    
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "dataframe_records": [{"query": query}]
    }
    
    response = requests.post(endpoint_url, headers=headers, json=payload)
    
    if response.status_code == 403:
        raise PermissionError("Access denied. Check token permissions.")
    
    response.raise_for_status()
    return response.json()

# Usage
try:
    result = query_endpoint_with_pat("rag-chatbot-endpoint", "What is Unity Catalog?")
    print(result)
except PermissionError as e:
    print(f"Permission error: {e}")
```

### OAuth Integration for User Apps

```python
from databricks.sdk.oauth import OAuthClient

# OAuth flow for user-facing applications
oauth_client = OAuthClient(
    host=os.getenv("DATABRICKS_HOST"),
    client_id=os.getenv("OAUTH_CLIENT_ID"),
    client_secret=os.getenv("OAUTH_CLIENT_SECRET"),
    redirect_url="http://localhost:8000/callback"
)

# Get authorization URL
auth_url = oauth_client.get_authorization_url(
    scopes=["sql", "serving-endpoints"]
)
print(f"Visit: {auth_url}")

# After user authorizes, exchange code for token
token = oauth_client.get_token_from_code(authorization_code)

# Use token to query endpoint
headers = {
    "Authorization": f"Bearer {token.access_token}",
    "Content-Type": "application/json"
}
```

### IP Allowlist Configuration

```python
# Configure via Databricks workspace settings (Admin Console)
# Or using API:

from databricks.sdk.service.settings import IpAccessList

# Add IP allowlist
w.ip_access_lists.create(
    label="production-servers",
    list_type="ALLOW",
    ip_addresses=[
        "203.0.113.0/24",  # Production server subnet
        "198.51.100.42/32"  # Specific production host
    ]
)

# Endpoint will only accept requests from these IPs
```

### Rate Limiting per User/Group

```python
from typing import Dict
import time

class RateLimiter:
    def __init__(self, requests_per_minute: int):
        self.limit = requests_per_minute
        self.requests: Dict[str, list] = {}
    
    def allow_request(self, user_id: str) -> bool:
        """Check if user is within rate limit."""
        
        now = time.time()
        
        if user_id not in self.requests:
            self.requests[user_id] = []
        
        # Remove requests older than 1 minute
        self.requests[user_id] = [
            req_time for req_time in self.requests[user_id]
            if now - req_time < 60
        ]
        
        # Check limit
        if len(self.requests[user_id]) >= self.limit:
            return False
        
        # Add new request
        self.requests[user_id].append(now)
        return True

# Wrapper for endpoint queries
rate_limiter = RateLimiter(requests_per_minute=100)

def query_with_rate_limit(user_id: str, query: str):
    if not rate_limiter.allow_request(user_id):
        raise Exception("Rate limit exceeded. Try again later.")
    
    # Proceed with query
    return query_endpoint_with_pat("rag-chatbot-endpoint", query)
```

### Audit Logging

```python
# Enable audit logs (workspace admin setting)
# Query audit logs for endpoint access

audit_logs_query = """
SELECT 
    timestamp,
    user_identity.email as user,
    request_params.endpoint_name as endpoint,
    response.status_code as status,
    request_params.query as query_preview
FROM system.access.audit
WHERE action_name = 'modelServing'
    AND request_params.endpoint_name = 'rag-chatbot-endpoint'
    AND timestamp >= current_date - INTERVAL 7 DAYS
ORDER BY timestamp DESC
"""

audit_df = spark.sql(audit_logs_query)
audit_df.display()
```

### Fine-Grained Access with Context

```python
class ContextualAccessControl:
    def __init__(self):
        self.user_permissions = {}
    
    def can_query_endpoint(self, user_id: str, endpoint: str, query_context: dict) -> bool:
        """Check if user can query endpoint with given context."""
        
        # Check user group
        user_groups = self.get_user_groups(user_id)
        
        # Check if user has endpoint access
        if "ml-team" not in user_groups and "data-science" not in user_groups:
            return False
        
        # Check data scope (e.g., can only query own department's data)
        if query_context.get("department") != self.get_user_department(user_id):
            return False
        
        return True
    
    def get_user_groups(self, user_id: str) -> list:
        # Fetch from Databricks API or cache
        return ["ml-team"]
    
    def get_user_department(self, user_id: str) -> str:
        # Fetch from user metadata
        return "engineering"

# Usage
access_control = ContextualAccessControl()

def secure_query(user_id: str, query: str, department: str):
    context = {"department": department}
    
    if not access_control.can_query_endpoint(user_id, "rag-chatbot-endpoint", context):
        raise PermissionError("Access denied based on query context")
    
    # Proceed with query
    return query_endpoint_with_pat("rag-chatbot-endpoint", query)
```

## Best Practices

- **Principle of least privilege**: Grant minimum necessary permissions.
- **Service principals for apps**: Use service principals instead of personal tokens for production apps.
- **Rotate tokens regularly**: Set expiration on tokens; rotate every 90 days.
- **Audit access**: Enable and monitor audit logs for endpoint access.
- **Network isolation**: Use private link or IP allowlists for production endpoints.
- **Rate limiting**: Implement per-user/app rate limits to prevent abuse.

## Sample Questions

1. How grant endpoint access to user group?
2. Difference between CAN_QUERY and CAN_MANAGE?
3. When use service principal vs user token?
4. How implement rate limiting?
5. Best practice for production endpoint access?

## Answers

1. Use `w.serving_endpoints.set_permissions()` with group_name and permission_level CAN_QUERY.
2. CAN_QUERY: can only call endpoint for inference; CAN_MANAGE: can update endpoint config, permissions, and delete.
3. Service principal for applications/services (non-interactive); user token for personal scripts/notebooks (interactive).
4. Track requests per user in time window (e.g., sliding 1-minute window); reject if exceeds threshold; use Redis/cache for distributed systems.
5. Use service principals with short-lived tokens, enable IP allowlists, implement rate limiting, audit logs, use Unity Catalog permissions, separate staging/prod endpoints.

## References

- [Databricks Model Serving Security](https://docs.databricks.com/en/machine-learning/model-serving/security.html)
- [Unity Catalog Privileges](https://docs.databricks.com/en/data-governance/unity-catalog/manage-privileges/index.html)

---

Previous: [Model Serving Endpoint Deployment](./03-model-serving-endpoint-deployment.md)  
Next: [Batch Inference with ai_query](./05-batch-inference-ai-query.md)
