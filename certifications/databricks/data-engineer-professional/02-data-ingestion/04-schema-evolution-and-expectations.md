# Schema Evolution and Expectations in Ingestion

## Overview

Manage schema drift with Auto Loader schema inference, Delta schema evolution modes, and enforce data quality with expectations/constraints.

## Prerequisites

- Auto Loader and Delta Lake basics
- Unity Catalog for table management

## Concepts

- Schema inference: Auto Loader `cloudFiles.schemaLocation` for drift tracking
- Schema evolution modes: `mergeSchema`, `overwriteSchema`
- Expectations: Inline quality checks in Lakeflow pipelines; quarantine bad records
- Constraints: `NOT NULL`, `CHECK` constraints on Delta tables

## Hands-on Walkthrough

### Auto Loader with Schema Evolution (Python)

```python
from pyspark.sql.functions import current_timestamp

(spark.readStream.format("cloudFiles")
 .option("cloudFiles.format", "json")
 .option("cloudFiles.schemaLocation", "/mnt/schemas/orders")
 .option("cloudFiles.inferColumnTypes", "true")
 .load("/mnt/raw/orders/")
 .withColumn("_ingest_ts", current_timestamp())
 .writeStream
 .option("checkpointLocation", "/mnt/ckpt/orders_bronze")
 .option("mergeSchema", "true")
 .trigger(availableNow=True)
 .toTable("prod.raw.orders_bronze"))
```

### Delta Merge Schema on Batch Write (Python)

```python
df = spark.read.parquet("/mnt/incoming/batch/")
df.write.format("delta") \
  .mode("append") \
  .option("mergeSchema", "true") \
  .saveAsTable("prod.raw.batch_bronze")
```

### Add Constraints (SQL)

```sql
ALTER TABLE prod.raw.orders_bronze
ADD CONSTRAINT valid_amount CHECK (amount >= 0);

ALTER TABLE prod.raw.orders_bronze
ALTER COLUMN order_id SET NOT NULL;
```

### Lakeflow Expectations (Python)

```python
import dlt

@dlt.table(
  name="orders_silver",
  table_properties={"quality": "silver"}
)
@dlt.expect_or_drop("valid_email", "email IS NOT NULL AND email RLIKE '^[^@]+@[^@]+\\.[^@]+$'")
@dlt.expect_or_fail("valid_country", "country IN ('US', 'CA', 'MX')")
def orders_silver():
    return spark.readStream.table("prod.raw.orders_bronze")
```

## Production Considerations

- Schema location: Centralize schema tracking; version schemas for rollback.
- Evolution strategy: Use `mergeSchema` for additive changes; plan major rewrites carefully.
- Expectations placement: Quarantine at Bronze, enforce at Silver.
- Monitoring: Track expectation violations via system tables (`system.liveflow.event_log`).

## Troubleshooting

- Incompatible schema changes: Types mismatch or column drops fail; use `overwriteSchema` cautiously.
- Constraint violations: Log violating records before dropping; alert on spikes.
- Inference delays: Schema inference can lag; seed initial schema for stability.

## Sample Questions

1. How does Auto Loader track schema changes?  
2. What is the difference between `mergeSchema` and `overwriteSchema`?  
3. When should expectations use `expect_or_drop` vs `expect_or_fail`?  
4. How do you monitor expectation violations at scale?  
5. What Delta feature enforces column constraints?

## Answers

1. Via `cloudFiles.schemaLocation`, persisting inferred schemas and tracking drift.  
2. `mergeSchema` adds new columns; `overwriteSchema` replaces the entire schema.  
3. `expect_or_drop` quarantines bad rows; `expect_or_fail` stops the pipeline on violations.  
4. Query `system.liveflow.event_log` for expectation metrics and alerts.  
5. Delta constraints (`CHECK`, `NOT NULL`).

## References

- [Auto Loader schema inference](https://docs.databricks.com/ingestion/auto-loader/schema.html)
- [Delta Lake schema evolution](https://docs.databricks.com/delta/schema-evolution.html)
- [Lakeflow expectations](https://docs.databricks.com/lakehouse/data-engineering/declare-transforms/expectations.html)

---

Previous: [Append-Only Pipelines](./03-append-only-pipelines.md)  
Next: [Quality Controls and Observability](./05-quality-controls-and-observability.md)
