# Debugging Lakeflow Declarative Pipelines and Spark Pipelines

## Overview

Lakeflow Declarative Pipelines and Spark pipelines expose rich runtime telemetry through event logs and Spark UI. This guide shows how to locate and interpret pipeline event logs, correlate with Spark UI, and apply fixes.

## Lakeflow Event Logs

- Each pipeline writes structured event logs (JSON-like records) to the pipeline storage location.  
- Events include: state changes, validation errors, updates, input/output lineage, expectations, and operation metrics.

### Reading event logs

```python
# Replace with your pipeline storage location
event_log_path = "/pipelines/storage/my_pipeline/event_log/"
events = spark.read.format("delta").load(event_log_path)

events.select("timestamp","level","message","flow_id","update_id").orderBy("timestamp", ascending=False).show(20, truncate=False)
```

```sql
-- Common queries
-- Recent errors
SELECT timestamp, level, message
FROM delta.`/pipelines/storage/my_pipeline/event_log/`
WHERE level IN ("ERROR","FATAL")
ORDER BY timestamp DESC
LIMIT 50;

-- Failed expectations or quality checks
SELECT timestamp, expectation, failed_records
FROM delta.`/pipelines/storage/my_pipeline/event_log/`
WHERE expectation IS NOT NULL
ORDER BY timestamp DESC;
```

## Correlating with Spark UI

1. Note `update_id` or run timestamp from event logs.  
2. Open Spark UI for the cluster that executed that run.  
3. In SQL tab, filter by time window; inspect physical plan for slow operators.  
4. Cross-check with event messages: e.g., slow sink, schema evolution, expectation failures.

## Common Issues and Fixes

- Ingestion lag with Auto Loader → enable notification mode, increase `maxFilesPerTrigger`, tune parallelism.
- Expectation failures → route bad records to quarantine volume/table, add schema hints.
- Schema drift → enable schema evolution (`mergeSchema`) with governance.
- Slow joins/aggregations → AQE, broadcast small dimensions, partition pruning.

```python
# Example: robust streaming write with compaction
(df.writeStream
  .format("delta")
  .option("checkpointLocation", "/Volumes/ops/metadata/checkpoints/orders")
  .option("mergeSchema", "true")
  .option("optimizeWrite", "true")
  .option("autoCompact", "true")
  .trigger(processingTime="5 minutes")
  .toTable("silver.orders")
)
```

## Debugging Checklist

- Event logs: identify failing component, expectation, or transformation.
- Spark UI: check for skew, spills, shuffle retries, and failed stages.
- Storage: verify checkpoint health and table VACUUM/OPTIMIZE cadence.
- Dependencies: confirm library versions pinned and compatible.

## Mock Questions

1. **Where do you find detailed pipeline state transitions and expectation outcomes?**  
    a. System access audit  
    b. Lakeflow event logs  
    c. Workspace browser history  
    d. Driver log only

2. **Which setting reduces small file problems for streaming sinks?**  
    a. `spark.sql.shuffle.partitions = 1`  
    b. `autoCompact` on writeStream  
    c. Disable checkpointing  
    d. Use CSV instead of Delta

3. **How do you correlate a failed pipeline update with Spark UI stages?**  
    a. Search by user email  
    b. Match `update_id`/timestamp from event logs to Spark UI run window  
    c. Check DBFS root for errors  
    d. It is not possible

## Answers

1. b
2. b
3. b

## References

- Lakeflow Declarative Pipelines event logs (Databricks docs)
- Structured Streaming troubleshooting (Databricks and Spark docs)
- Delta Lake optimize/auto compact best practices
