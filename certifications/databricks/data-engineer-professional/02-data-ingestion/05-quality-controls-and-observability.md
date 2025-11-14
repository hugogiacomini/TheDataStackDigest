# Quality Controls and Observability in Ingestion

## Overview

Implement end-to-end quality gates: validation rules, anomaly detection, alerting, and lineage for ingestion pipelines.

## Prerequisites

- Lakeflow or Structured Streaming pipelines
- Unity Catalog system tables access

## Concepts

- Data quality dimensions: Completeness, accuracy, consistency, timeliness
- Observability: Metrics (volume, latency), logs (event_log), lineage
- Alerting: SQL Alerts on system tables; webhooks for external integrations

## Hands-on Walkthrough

### Record-Level Validation (Python)

```python
from pyspark.sql.functions import col, length, regexp_extract

validated_df = (
    raw_df
    .withColumn("is_valid_email", col("email").rlike("^[^@]+@[^@]+\\.[^@]+$"))
    .withColumn("is_valid_zip", length(col("zip_code")) == 5)
)

quarantine_df = validated_df.filter(~(col("is_valid_email") & col("is_valid_zip")))
clean_df = validated_df.filter(col("is_valid_email") & col("is_valid_zip"))

quarantine_df.write.mode("append").saveAsTable("prod.quarantine.orders_invalid")
clean_df.write.mode("append").saveAsTable("prod.raw.orders_clean")
```

### Lakeflow Metrics Tracking (Python)

```python
import dlt

@dlt.table
@dlt.expect("valid_amount", "amount > 0")
def orders_validated():
    return dlt.read("orders_bronze")

# Metrics accessible via system.liveflow.event_log
```

### SQL Alert for Volume Anomaly (SQL)

```sql
-- Create alert to detect ingestion drops
CREATE ALERT orders_ingestion_drop
SCHEDULE CRON '0 * * * *'
AS
SELECT COUNT(*) AS row_count, MAX(_ingest_ts) AS last_ingest
FROM prod.raw.orders_bronze
WHERE _ingest_ts >= current_timestamp() - INTERVAL 1 HOUR
HAVING COUNT(*) < 1000;
```

### Query Event Log for Quality Metrics (SQL)

```sql
SELECT event_type, details:flow_name, details:dataset_name,
       details:expectations, timestamp
FROM system.liveflow.event_log
WHERE details:flow_name = 'orders_pipeline'
  AND event_type = 'flow_progress'
ORDER BY timestamp DESC
LIMIT 100;
```

## Production Considerations

- Alert fatigue: Tune thresholds; use statistical baselines (e.g., 3-sigma).
- Lineage integration: Link quarantine records back to source files for debugging.
- SLA tracking: Monitor end-to-end latency from file arrival to Silver availability.
- Cost: Balance granular validation with compute overhead; sample when feasible.

## Troubleshooting

- Silent data quality degradation: Establish baseline metrics and trend analysis.
- False positives: Refine validation rules iteratively with domain input.
- Missing lineage: Ensure `_ingest_file` or metadata columns are preserved.

## Sample Questions

1. What are the core dimensions of data quality?  
2. How can you detect volume anomalies in ingestion?  
3. Where are Lakeflow expectation metrics stored?  
4. Why quarantine invalid records instead of dropping them?  
5. How do you alert on ingestion SLA breaches?

## Answers

1. Completeness, accuracy, consistency, timeliness.  
2. SQL Alerts on row counts per time window; detect drops below baseline.  
3. `system.liveflow.event_log`.  
4. To diagnose upstream issues and prevent silent data loss.  
5. Create SQL Alerts monitoring latency between file arrival and table updates.

## References

- [SQL Alerts](https://docs.databricks.com/sql/user/alerts/index.html)
- [System tables](https://docs.databricks.com/administration-guide/system-tables/index.html)
- [Lakeflow event log](https://docs.databricks.com/lakehouse/data-engineering/declare-transforms/event-log.html)

---

Previous: [Schema Evolution and Expectations](./04-schema-evolution-and-expectations.md)
