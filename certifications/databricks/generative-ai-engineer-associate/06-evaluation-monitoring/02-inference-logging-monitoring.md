# Inference Logging and Performance Monitoring

## Overview

Use inference tables and Agent Monitoring to track live LLM endpoint performance: request/response logging, quality metrics, cost analysis, and issue detection.

## Inference Tables

**Automatic Logging**: Databricks Model Serving logs all requests/responses to Delta tables in Unity Catalog.

**Schema**:

```text
- request_id (string)
- timestamp (timestamp)
- request (struct: inputs, parameters)
- response (struct: predictions, metadata)
- latency_ms (int)
- status_code (int)
- endpoint_name (string)
```

## Hands-on Examples

### Enable Inference Logging

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import ServedEntityInput, EndpointCoreConfigInput

w = WorkspaceClient()

# Create endpoint with inference logging
w.serving_endpoints.create(
    name="rag_chatbot_endpoint",
    config=EndpointCoreConfigInput(
        served_entities=[
            ServedEntityInput(
                entity_name="main.ml_models.rag_chatbot",
                entity_version="1",
                scale_to_zero_enabled=True
            )
        ],
        # Enable inference logging
        auto_capture_config={
            "catalog_name": "main",
            "schema_name": "monitoring",
            "table_name_prefix": "rag_inference"
        }
    )
)
```

### Query Inference Table

```python
# Inference table name: {catalog}.{schema}.{prefix}_{endpoint_name}
inference_table = "main.monitoring.rag_inference_rag_chatbot_endpoint"

# Analyze request patterns
request_stats = spark.sql(f"""
SELECT 
    DATE(timestamp) as date,
    COUNT(*) as total_requests,
    AVG(latency_ms) as avg_latency,
    PERCENTILE(latency_ms, 0.95) as p95_latency,
    COUNT(CASE WHEN status_code != 200 THEN 1 END) as error_count
FROM {inference_table}
WHERE timestamp >= CURRENT_DATE - INTERVAL 7 DAYS
GROUP BY DATE(timestamp)
ORDER BY date DESC
""")

request_stats.display()
```

### Extract and Analyze Responses

```python
# Parse request/response JSON
from pyspark.sql.functions import col, from_json, schema_of_json

# Sample row to infer schema
sample = spark.table(inference_table).select("response").first()
response_schema = schema_of_json(sample.response)

# Extract answer quality indicators
quality_df = spark.sql(f"""
SELECT 
    request_id,
    timestamp,
    request.inputs[0].query as user_query,
    response.predictions[0].answer as llm_answer,
    response.predictions[0].sources as sources,
    latency_ms,
    LENGTH(response.predictions[0].answer) as answer_length
FROM {inference_table}
WHERE timestamp >= CURRENT_DATE - INTERVAL 1 DAY
""")

quality_df.display()
```

### Detect Quality Issues

```python
# Identify potential hallucinations (short answers, no sources)
problematic_responses = spark.sql(f"""
SELECT 
    request_id,
    timestamp,
    request.inputs[0].query as query,
    response.predictions[0].answer as answer,
    SIZE(response.predictions[0].sources) as source_count,
    latency_ms
FROM {inference_table}
WHERE timestamp >= CURRENT_DATE - INTERVAL 1 DAY
    AND (
        LENGTH(response.predictions[0].answer) < 50  -- Too short
        OR SIZE(response.predictions[0].sources) = 0  -- No citations
    )
ORDER BY timestamp DESC
""")

problematic_responses.display()
```

## Agent Monitoring

**Databricks Agent Monitoring**: Specialized monitoring for AI agents with multi-step reasoning.

### Configure Agent Monitoring

```python
from databricks.agents import AgentMonitor

# Initialize monitor
monitor = AgentMonitor(
    model_name="main.ml_models.customer_support_agent",
    inference_table="main.monitoring.agent_inference"
)

# Define quality checks
monitor.add_check(
    name="response_completeness",
    metric="answer_length",
    threshold=100,
    operator=">"
)

monitor.add_check(
    name="citation_presence",
    metric="source_count",
    threshold=1,
    operator=">="
)

# Enable monitoring
monitor.enable()
```

### Analyze Agent Traces

```python
# Agent trace includes tool calls, reasoning steps
agent_traces = spark.sql(f"""
SELECT 
    request_id,
    timestamp,
    request.inputs[0].query as user_query,
    response.predictions[0].tool_calls as tools_used,
    response.predictions[0].reasoning_steps as steps,
    response.predictions[0].final_answer as answer,
    latency_ms
FROM main.monitoring.agent_inference
WHERE timestamp >= CURRENT_DATE - INTERVAL 7 DAYS
""")

# Count tool usage
tool_usage = spark.sql(f"""
SELECT 
    EXPLODE(response.predictions[0].tool_calls) as tool_call,
    COUNT(*) as usage_count
FROM main.monitoring.agent_inference
WHERE timestamp >= CURRENT_DATE - INTERVAL 7 DAYS
GROUP BY tool_call
ORDER BY usage_count DESC
""")
```

## Cost Tracking

```python
# Calculate token usage and cost
cost_analysis = spark.sql(f"""
SELECT 
    DATE(timestamp) as date,
    COUNT(*) as total_requests,
    SUM(response.predictions[0].usage.prompt_tokens) as total_prompt_tokens,
    SUM(response.predictions[0].usage.completion_tokens) as total_completion_tokens,
    SUM(response.predictions[0].usage.total_tokens) as total_tokens,
    -- Assuming $0.001 per 1K tokens (adjust based on model)
    SUM(response.predictions[0].usage.total_tokens) * 0.001 / 1000 as estimated_cost_usd
FROM {inference_table}
WHERE timestamp >= CURRENT_DATE - INTERVAL 30 DAYS
GROUP BY DATE(timestamp)
ORDER BY date DESC
""")

cost_analysis.display()
```

## Alerting and Dashboards

```python
# Create monitoring dashboard
from databricks.sdk.service.sql import Query, Visualization

# Define alert query
alert_query = f"""
SELECT 
    COUNT(*) as error_count
FROM {inference_table}
WHERE timestamp >= CURRENT_TIMESTAMP - INTERVAL 1 HOUR
    AND status_code != 200
"""

# Create alert (using Databricks SQL Alerts)
# w.alerts.create(...)
```

## Best Practices

- **Retention policy**: Set TTL on inference tables (e.g., 90 days) to manage storage.
- **Sampling**: For high-volume endpoints, sample requests for detailed analysis.
- **PII scrubbing**: Mask sensitive data in logged requests (configure in endpoint).
- **Aggregate metrics**: Pre-compute hourly/daily metrics for dashboard performance.
- **Anomaly detection**: Set up alerts for latency spikes, error rate increases.
- **Cost budgets**: Track token usage trends; set budget alerts.

## Sample Questions

1. What is an inference table?
2. How enable inference logging?
3. Typical latency metric to monitor?
4. How detect hallucinations in logs?
5. Difference between inference table and Agent Monitoring?

## Answers

1. Delta table auto-populated with model serving requests/responses for monitoring.
2. Set `auto_capture_config` with catalog/schema/table prefix when creating endpoint.
3. P95 latency (95th percentile); ensures most users experience acceptable performance.
4. Check for short answers with no sources, or compare answer against retrieved context semantically.
5. Inference table logs all requests; Agent Monitoring adds agent-specific traces (tool calls, reasoning steps).

## References

- [Databricks Model Serving Monitoring](https://docs.databricks.com/en/machine-learning/model-serving/inference-tables.html)
- [Agent Monitoring](https://docs.databricks.com/en/generative-ai/agent-evaluation/monitoring.html)

---

Previous: [RAG Evaluation with MLflow](./01-rag-evaluation-mlflow.md)
