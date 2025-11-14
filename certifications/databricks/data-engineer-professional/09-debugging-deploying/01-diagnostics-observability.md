# Diagnostics and Observability: Spark UI, Cluster Logs, System Tables, Query Profiles

## Overview

Effective debugging starts with knowing where and how to observe. Databricks provides multiple telemetry surfaces that complement each other:

- Spark UI for stage/task plans, shuffles, and skew.
- Query Profile for SQL query–level operators and metrics.
- Cluster/driver/executor logs for JVM/Python errors and library/runtime issues.
- Unity Catalog system tables for historical observability, audit, and cost.

This guide shows what to look for, where to find it, and actionable steps to remediate performance and reliability issues.

## Spark UI Essentials

- Stages and tasks: identify data skew via outlier task durations and input sizes.
- DAG visualization: confirm predicate pushdown, join order, and shuffle boundaries.
- SQL tab: analyze physical plan operators (BroadcastHashJoin, SortMergeJoin, Exchange).
- Storage tab: verify caching/persisting behavior.

### Quick workflow

1. Reproduce the issue with a deterministic job input (e.g., fixed date partition).
2. Open Spark UI → SQL tab; locate the slow query by start time and duration.
3. Inspect Exchange (shuffle) nodes, skewed partitions, and join strategies.
4. Cross-check with Query Profile for operator-level metrics.

## Query Profile (SQL) Essentials

Query Profile provides a concise tree of operators with metrics like rows, time, spill, and codegen.

### Typical fixes from Query Profile

- Switch joins: prefer BroadcastHashJoin when one side < broadcast threshold.
- Reduce skew: salting keys, AQE skew join handling, or re-partitioning.
- Reduce spills: increase shuffle partitions or executor memory, or optimize file sizes.

```sql
-- Enable adaptive query execution and broadcast where safe
SET spark.sql.adaptive.enabled = true;
SET spark.sql.autoBroadcastJoinThreshold = 50MB;
```

## Cluster Logs: Where and What

- Driver logs: end-to-end job errors, Python tracebacks, library resolution issues.
- Executor logs: task-level exceptions, OOM, GC pressure, native I/O errors.

### Accessing logs

- UI: Compute → Your Cluster → Logs (Driver/Executor).  
- Files: `dbfs:/logs/` or workspace logs if configured.

```python
# Example: fetch driver log snippets for quick inspection
for f in dbutils.fs.ls("dbfs:/logs/"):
    if f.name.endswith("driver.log"):
        print("Reading:", f.path)
        print(dbutils.fs.head(f.path, 10000))
```

## System Tables for Observability (Unity Catalog)

Use system tables to analyze performance, cost, and access at scale.

```sql
-- Recently expensive SQL queries
SELECT
  query_id,
  user_identity.email AS user_email,
  total_duration_ms,
  billed_bytes,
  start_time
FROM system.query.history
WHERE start_time >= date_sub(current_timestamp(), 7)
ORDER BY billed_bytes DESC
LIMIT 50;

-- Job run outcomes (example; adjust to your environment)
SELECT *
FROM system.workflow.jobs
WHERE start_time >= date_sub(current_timestamp(), 7)
ORDER BY start_time DESC;

-- Access audit for sensitive resources
SELECT event_time, user_identity.email, action_name, request_params.path
FROM system.access.audit
WHERE event_date >= current_date() - 7
ORDER BY event_time DESC;
```

## Common Failure Modes and Fixes

- Executor OOM → Increase partition count, reduce coalesce before wide ops, use `persist(StorageLevel.DISK_ONLY)` when memory pressure is high.
- Skewed joins → Enable AQE, use salting, or pre-aggregate before join.
- Excessive small files → Compact via OPTIMIZE or repartition before write (optimizeWrite/autoCompact for streaming).
- UDF performance → Prefer built-in functions; use Pandas UDF only when necessary; ensure vectorization.

```python
# Example: mitigate skew with salting
from pyspark.sql.functions import col, monotonically_increasing_id

salt_buckets = 16
left_salted = left_df.withColumn("__salt", (monotonically_increasing_id() % salt_buckets))
right_salted = right_df.withColumn("__salt", (monotonically_increasing_id() % salt_buckets))
joined = left_salted.join(right_salted, ["join_key", "__salt"], "inner").drop("__salt")
```

## Production Considerations

- Turn on AQE in production clusters; validate with representative workloads.
- Size clusters for shuffle workloads (I/O bandwidth often dominates CPU).
- Keep dependencies lean; pin versions to avoid runtime drift.
- Use managed tables and optimized file sizes (128–512 MB) for read efficiency.

## Mock Questions

1. **Which telemetry surface best reveals skewed partitions in a shuffle?**  
    a. Cluster driver logs  
    b. Spark UI Stages/Tasks  
    c. Query Profile  
    d. System access audit

2. **Given frequent spills in SortMergeJoin, what is the first optimization to try?**  
    a. Disable AQE  
    b. Reduce broadcast threshold to 0  
    c. Increase shuffle partitions or memory  
    d. Convert to CSV

3. **Which system table is the best starting point to enumerate expensive queries?**  
    a. system.access.audit  
    b. system.query.history  
    c. system.workflow.jobs  
    d. system.billing.costs

## Answers

1. b
2. c
3. b

## References

- Spark UI and SQL details: Databricks docs (Spark UI, Query Profile)
- System tables: Databricks system tables (query history, audit)
- Adaptive Query Execution (AQE): Apache Spark SQL tuning
