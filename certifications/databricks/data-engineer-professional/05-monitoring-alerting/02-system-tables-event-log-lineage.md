# System Tables, Event Logs, and Lineage

## Overview

Leverage Unity Catalog system tables for operational insights: billing, audit, lineage, and event logs for pipeline diagnostics.

## Prerequisites

- Unity Catalog metastore admin or usage privileges
- Access to `system` catalog

## Concepts

- System tables: `system.billing.usage`, `system.access.audit`, `system.compute.clusters`
- Lakeflow event log: `system.liveflow.event_log` for pipeline metrics
- Lineage: Table and column-level dependencies via Unity Catalog API and UI

## Hands-on Walkthrough

### Query Billing Usage (SQL)

```sql
SELECT usage_date, sku_name, SUM(usage_quantity) AS total_dbu
FROM system.billing.usage
WHERE usage_date >= current_date() - INTERVAL 7 DAYS
GROUP BY usage_date, sku_name
ORDER BY usage_date DESC, total_dbu DESC;
```

### Audit Log for Data Access (SQL)

```sql
SELECT event_time, user_identity.email, request_params.full_name_arg AS table_name,
       action_name
FROM system.access.audit
WHERE action_name IN ('getTable', 'readTable')
  AND event_date >= current_date() - INTERVAL 1 DAY
ORDER BY event_time DESC
LIMIT 100;
```

### Lakeflow Event Log for Pipeline Health (SQL)

```sql
SELECT timestamp, event_type, details:flow_name, details:dataset_name,
       details:num_output_rows, details:expectations
FROM system.liveflow.event_log
WHERE details:flow_name = 'retail_pipeline'
  AND event_type IN ('flow_progress', 'user_action')
ORDER BY timestamp DESC
LIMIT 50;
```

### Lineage via Unity Catalog (Python)

```python
# Retrieve lineage via Unity Catalog API or UI
# Example: Use Databricks REST API to fetch upstream/downstream dependencies
import requests

workspace_url = "https://<workspace>.cloud.databricks.com"
token = dbutils.secrets.get(scope="my_scope", key="token")

response = requests.get(
    f"{workspace_url}/api/2.0/unity-catalog/lineage-by-table",
    headers={"Authorization": f"Bearer {token}"},
    params={"table_name": "prod.retail.orders_silver"}
)
lineage = response.json()
print(lineage)
```

## Production Considerations

- Retention: System tables have retention policies; export for long-term analysis.
- Access control: Grant `SELECT` on `system` catalog schemas selectively.
- Cost monitoring: Schedule daily billing aggregations; alert on anomalies.
- Lineage impact analysis: Use before schema changes to identify downstream breakage.

## Troubleshooting

- Missing audit events: Ensure Unity Catalog is enabled and audit log delivery configured.
- Empty event log: Verify Lakeflow pipelines are configured with proper settings.
- Lineage gaps: Column-level lineage requires explicit column references in SQL/DataFrame ops.

## Sample Questions

1. Which system table tracks compute usage and costs?  
2. How do you audit who accessed a specific table?  
3. Where are Lakeflow pipeline metrics stored?  
4. What is column-level lineage used for?  
5. How can you detect cost spikes proactively?

## Answers

1. `system.billing.usage`.  
2. Query `system.access.audit` filtering by `action_name` and table name.  
3. `system.liveflow.event_log`.  
4. Impact analysis for schema changes and compliance tracking.  
5. Schedule SQL Alerts on `system.billing.usage` aggregations; set thresholds.

## References

- [System tables overview](https://docs.databricks.com/administration-guide/system-tables/index.html)
- [Audit logs](https://docs.databricks.com/administration-guide/system-tables/audit-logs.html)
- [Lineage in Unity Catalog](https://docs.databricks.com/data-governance/unity-catalog/data-lineage.html)

---

Previous: [Monitoring and Alerting](./01-monitoring-alerting.md)
