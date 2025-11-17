# Incremental Modelling Strategies

## Overview

Design incremental pipelines for large models: idempotency, checkpointing, CDF, and merge patterns across Bronze/Silver/Gold.

## Prerequisites

- Auto Loader or Lakeflow familiarity
- Delta Lake MERGE and CDF concepts

## Concepts

- Idempotent increments: Deterministic reprocessing using watermarks + checkpoints
- CDC: Use CDF or `APPLY CHANGES INTO` for declarative updates
- Backfills: Bounded replays with partition predicates

## Hands-on Walkthrough

### Auto Loader to Bronze (Python)

```python
(spark.readStream.format("cloudFiles")
 .option("cloudFiles.format", "json")
 .load("/mnt/raw/payments/")
 .writeStream
 .option("checkpointLocation", "/mnt/ckpt/payments_bronze")
 .trigger(availableNow=True)
 .toTable("prod.raw.payments_bronze"))
```

### Silver Merge from CDF (SQL)

```sql
ALTER TABLE prod.raw.payments_bronze SET TBLPROPERTIES (delta.enableChangeDataFeed = true);

MERGE INTO prod.curated.payments_silver s
USING (
  SELECT * FROM table_changes('prod.raw.payments_bronze', 'latest')
) c
ON s.payment_id = c.payment_id
WHEN MATCHED AND c._change_type IN ('update_postimage', 'update', 'delete') THEN UPDATE SET *
WHEN NOT MATCHED AND c._change_type IN ('insert', 'update_postimage') THEN INSERT *;
```

### Gold Aggregates Incrementally (SQL)

```sql
CREATE OR REPLACE TABLE prod.serving.daily_payments AS
SELECT date_trunc('day', payment_ts) AS day, SUM(amount) AS total
FROM prod.curated.payments_silver
GROUP BY 1;
```

## Production Considerations

- Watermarks: Track processed high-watermark per table.
- Checkpoints: Isolate per-stream job; avoid sharing.
- Promotion: Use Bundles to run the same plan across envs.
- Reorg: Optimize/`VACUUM` after large backfills.

## Troubleshooting

- Missed changes: Validate CDF retention; increase retention or snapshot backfill.
- Duplicates: Ensure deterministic keys and exactly-once sink semantics.
- Long-running merges: Batch by partition and `OPTIMIZE ZORDER` hot columns.

## Sample Questions

1. What makes an incremental pipeline idempotent?  
2. How does CDF simplify incremental processing?  
3. Why isolate checkpoints per job?  
4. How to safely backfill a subset of history?

## Answers

1. Deterministic MERGEs with tracked watermarks and checkpoints.  
2. Provides a table-native change stream for inserts/updates/deletes.  
3. Prevents cross-job interference and state corruption.  
4. Use partition predicates and replay only the target window.

## References

- [Change Data Feed](https://docs.databricks.com/delta/change-data-feed.html)
- [APPLY CHANGES INTO](https://docs.databricks.com/lakehouse/data-engineering/declare-transforms/apply-changes-into.html)

---

Previous: [Dimension and Fact Design](./04-dimension-fact-design-lakehouse.md)
