# Sequencing Endpoint Deployment Steps for RAG Applications

## Overview

Deploy RAG applications to production using Databricks Model Serving: register models to Unity Catalog, create serving endpoints, configure scaling, and enable monitoring.

## Deployment Sequence

```text
1. Register Model to Unity Catalog (MLflow)
2. Create Serving Endpoint
3. Configure Endpoint (resources, autoscaling)
4. Test Endpoint
5. Enable Monitoring (inference tables)
6. Promote to Production
```

## Hands-on Examples

### Step 1: Register Model to Unity Catalog

```python
import mlflow
from mlflow.models import infer_signature
import pandas as pd

# Assuming RAG model already developed as pyfunc
from rag_model import RAGPyfuncModel

# Set Unity Catalog as registry
mlflow.set_registry_uri("databricks-uc")

# Example input/output for signature
input_example = pd.DataFrame({"query": ["What is Unity Catalog?"]})
output_example = [{
    "answer": "Unity Catalog is a unified governance solution...",
    "sources": ["doc1.pdf", "doc2.pdf"]
}]

# Log and register model
with mlflow.start_run(run_name="rag_chatbot_v1") as run:
    # Log model
    mlflow.pyfunc.log_model(
        artifact_path="rag_model",
        python_model=RAGPyfuncModel(),
        signature=infer_signature(input_example, output_example),
        input_example=input_example,
        pip_requirements=[
            "langchain==0.1.0",
            "databricks-vectorsearch==0.22",
            "databricks-sdk>=0.18.0",
            "tiktoken==0.5.1"
        ]
    )
    
    # Log parameters
    mlflow.log_params({
        "embedding_model": "databricks-bge-large-en",
        "llm_endpoint": "databricks-meta-llama-3-1-70b-instruct",
        "chunk_size": 500,
        "top_k": 5
    })
    
    run_id = run.info.run_id

# Register to Unity Catalog
model_name = "main.ml_models.rag_chatbot"
model_version = mlflow.register_model(
    model_uri=f"runs:/{run_id}/rag_model",
    name=model_name,
    tags={"stage": "development", "team": "data-science"}
)

print(f"Registered {model_name} version {model_version.version}")
```

### Step 2: Create Serving Endpoint

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import (
    ServedEntityInput,
    EndpointCoreConfigInput,
    AutoCaptureConfigInput
)

w = WorkspaceClient()

endpoint_name = "rag-chatbot-endpoint"

# Create endpoint
endpoint = w.serving_endpoints.create(
    name=endpoint_name,
    config=EndpointCoreConfigInput(
        served_entities=[
            ServedEntityInput(
                entity_name=f"{model_name}",
                entity_version=str(model_version.version),
                workload_size="Small",  # Small, Medium, Large
                scale_to_zero_enabled=True,
                min_provisioned_throughput=0,  # Scale to zero
                max_provisioned_throughput=100  # Max requests/sec
            )
        ],
        # Enable inference logging
        auto_capture_config=AutoCaptureConfigInput(
            catalog_name="main",
            schema_name="monitoring",
            table_name_prefix="rag_inference"
        )
    )
)

print(f"Created endpoint: {endpoint_name}")

# Wait for endpoint to be ready
w.serving_endpoints.wait_get_serving_endpoint_not_updating(endpoint_name)
print("Endpoint is ready!")
```

### Step 3: Configure Autoscaling

```python
from databricks.sdk.service.serving import EndpointCoreConfigInput, ServedEntityInput

# Update endpoint with autoscaling
w.serving_endpoints.update_config(
    name=endpoint_name,
    served_entities=[
        ServedEntityInput(
            entity_name=f"{model_name}",
            entity_version=str(model_version.version),
            workload_size="Medium",
            scale_to_zero_enabled=False,  # Always available
            min_provisioned_throughput=10,  # Min 10 req/sec
            max_provisioned_throughput=200  # Max 200 req/sec
        )
    ]
)

print("Updated endpoint configuration")
```

### Step 4: Test Endpoint

```python
import requests
import json
import time

# Get endpoint URL
endpoint_info = w.serving_endpoints.get(endpoint_name)
endpoint_url = f"https://{w.config.host}/serving-endpoints/{endpoint_name}/invocations"

# Prepare request
headers = {
    "Authorization": f"Bearer {w.config.token}",
    "Content-Type": "application/json"
}

test_query = {
    "dataframe_records": [
        {"query": "What are the key features of Delta Lake?"}
    ]
}

# Test request
response = requests.post(
    endpoint_url,
    headers=headers,
    json=test_query,
    timeout=60
)

if response.status_code == 200:
    result = response.json()
    print("Test successful!")
    print(f"Answer: {result['predictions'][0]['answer']}")
    print(f"Sources: {result['predictions'][0]['sources']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

### Step 5: Enable Monitoring Dashboard

```python
# Query inference table
inference_table = "main.monitoring.rag_inference_rag_chatbot_endpoint"

# Create monitoring query
monitoring_query = f"""
SELECT 
    DATE(timestamp) as date,
    COUNT(*) as request_count,
    AVG(latency_ms) as avg_latency_ms,
    PERCENTILE(latency_ms, 0.95) as p95_latency_ms,
    COUNT(CASE WHEN status_code != 200 THEN 1 END) as error_count
FROM {inference_table}
WHERE timestamp >= CURRENT_DATE - INTERVAL 7 DAYS
GROUP BY DATE(timestamp)
ORDER BY date DESC
"""

monitoring_df = spark.sql(monitoring_query)
monitoring_df.display()
```

### Step 6: Promote to Production (Update Alias)

```python
from mlflow import MlflowClient

client = MlflowClient()

# Set production alias
client.set_registered_model_alias(
    name=model_name,
    alias="production",
    version=model_version.version
)

# Update endpoint to use production alias
w.serving_endpoints.update_config(
    name=endpoint_name,
    served_entities=[
        ServedEntityInput(
            entity_name=f"{model_name}",
            entity_version="production",  # Use alias instead of version number
            workload_size="Medium",
            scale_to_zero_enabled=False
        )
    ]
)

print(f"Promoted version {model_version.version} to production")
```

### Complete Deployment Script

```python
def deploy_rag_model(
    model_name: str,
    model_version: int,
    endpoint_name: str,
    workload_size: str = "Medium"
) -> dict:
    """Complete deployment workflow."""
    
    w = WorkspaceClient()
    
    # Check if endpoint exists
    try:
        existing_endpoint = w.serving_endpoints.get(endpoint_name)
        print(f"Endpoint {endpoint_name} already exists. Updating...")
        
        # Update with new version
        w.serving_endpoints.update_config(
            name=endpoint_name,
            served_entities=[
                ServedEntityInput(
                    entity_name=model_name,
                    entity_version=str(model_version),
                    workload_size=workload_size,
                    scale_to_zero_enabled=False
                )
            ]
        )
        action = "updated"
        
    except Exception:
        print(f"Creating new endpoint {endpoint_name}...")
        
        # Create new endpoint
        w.serving_endpoints.create(
            name=endpoint_name,
            config=EndpointCoreConfigInput(
                served_entities=[
                    ServedEntityInput(
                        entity_name=model_name,
                        entity_version=str(model_version),
                        workload_size=workload_size,
                        scale_to_zero_enabled=True
                    )
                ],
                auto_capture_config=AutoCaptureConfigInput(
                    catalog_name="main",
                    schema_name="monitoring",
                    table_name_prefix="inference"
                )
            )
        )
        action = "created"
    
    # Wait for ready
    print("Waiting for endpoint to be ready...")
    w.serving_endpoints.wait_get_serving_endpoint_not_updating(endpoint_name)
    
    # Test endpoint
    print("Testing endpoint...")
    endpoint_url = f"https://{w.config.host}/serving-endpoints/{endpoint_name}/invocations"
    
    test_response = requests.post(
        endpoint_url,
        headers={
            "Authorization": f"Bearer {w.config.token}",
            "Content-Type": "application/json"
        },
        json={"dataframe_records": [{"query": "Test query"}]},
        timeout=60
    )
    
    return {
        "endpoint_name": endpoint_name,
        "action": action,
        "status": "success" if test_response.status_code == 200 else "failed",
        "model_version": model_version
    }

# Usage
result = deploy_rag_model(
    model_name="main.ml_models.rag_chatbot",
    model_version=3,
    endpoint_name="rag-chatbot-production",
    workload_size="Medium"
)
print(result)
```

## Best Practices

- **Use Unity Catalog aliases**: Deploy with "production"/"staging" aliases, not version numbers.
- **Test before production**: Deploy to staging endpoint first; validate thoroughly.
- **Gradual rollout**: Use traffic splitting between versions (A/B testing).
- **Monitor inference logs**: Track latency, errors, token usage from day one.
- **Versioning**: Tag models with metadata (git SHA, training date, metrics).
- **Rollback plan**: Keep previous version ready; quick revert via alias update.

## Sample Questions

1. What is model registration step?
2. How configure endpoint autoscaling?
3. Difference between version number and alias?
4. How test endpoint before production?
5. Best practice for rollback?

## Answers

1. Save model to Unity Catalog with MLflow; creates versioned artifact with metadata and dependencies.
2. Set `min_provisioned_throughput` and `max_provisioned_throughput` in `ServedEntityInput`; choose `workload_size` (Small/Medium/Large).
3. Version number is immutable (1, 2, 3); alias is mutable pointer ("production", "staging") that can be updated without endpoint changes.
4. Create separate staging endpoint; run test queries; validate latency/accuracy; check inference logs before promoting.
5. Use aliases; keep previous version deployed; switch alias back in seconds; monitor metrics after any change.

## References

- [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html)
- [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html)

---

Previous: [Vector Search Index Creation](./02-vector-search-index-creation.md)  
Next: [Access Control for Endpoints](./04-endpoint-access-control.md)
