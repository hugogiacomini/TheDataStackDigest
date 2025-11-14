# Skew Handling and Wide Transformation Optimization

## Overview

Diagnose and mitigate data skew in joins/aggregations; optimize wide transformations with salting, bucketing, and Adaptive Query Execution (AQE).

## Prerequisites

- Spark execution model (stages, tasks, shuffles)
- Familiarity with DataFrames and SQL plans

## Concepts

- Skew: Uneven partition sizes causing straggler tasks
- Salting: Add random prefix to skewed keys; post-aggregate to merge
- AQE skew join optimization: Automatically splits large partitions
- Bucketing: Pre-shuffle tables on join keys to avoid shuffle

## Hands-on Walkthrough

### Detect Skew via Spark UI

- Navigate to Spark UI → Stages → Task metrics
- Identify tasks with max duration >> median (straggler symptom)
- Check shuffle read sizes per task for imbalance

### Enable AQE Skew Join Optimization (Python)

```python
# Enable Adaptive Query Execution
spark.conf.set("spark.sql.adaptive.enabled", "true")

# Enable AQE skew join optimization to split large partitions
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# Set threshold for partition size to be considered skewed (default 256MB)
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256MB")

# AQE will split large partitions during join automatically
result = large_df.join(skewed_df, "key_col")
```

### Manual Salting for Aggregations (Python)

```python
from pyspark.sql.functions import col, lit, concat, rand, floor

salted_df = (
    df.withColumn("salt", floor(rand() * 10).cast("int"))
      .withColumn("salted_key", concat(col("key_col"), lit("_"), col("salt")))
)

# Aggregate on salted key
partial_agg = salted_df.groupBy("salted_key").agg({"value": "sum"})

# Remove salt and final aggregate
final_agg = (
    partial_agg
    .withColumn("key_col", regexp_extract(col("salted_key"), "^(.+)_\\d+$", 1))
    .groupBy("key_col")
    .agg({"sum(value)": "sum"})
)
```

### Bucketing for Repeated Joins (SQL)

```sql
CREATE TABLE prod.curated.orders_bucketed
USING DELTA
CLUSTERED BY (customer_id) INTO 32 BUCKETS
AS SELECT * FROM prod.curated.orders_silver;

CREATE TABLE prod.curated.customers_bucketed
USING DELTA
CLUSTERED BY (customer_id) INTO 32 BUCKETS
AS SELECT * FROM prod.curated.customers_silver;

-- Join avoids shuffle when buckets align
SELECT o.*, c.country
FROM prod.curated.orders_bucketed o
JOIN prod.curated.customers_bucketed c ON o.customer_id = c.customer_id;
```

## Production Considerations

- AQE first: Enable AQE before manual interventions; it handles many skew cases.
- Salting cost: Adds shuffle overhead; profile before/after.
- Bucketing maintenance: Requires rewrite on key changes; best for stable dimensions.
- Broadcast joins: Prefer for small dimension tables; set `spark.sql.autoBroadcastJoinThreshold`.

## Troubleshooting

- Skew persists after AQE: Manually salt or isolate hot keys.
- Bucket pruning issues: Verify bucket column is join key; check physical plan.
- Over-salting: Too many salts increases shuffle; balance by data distribution.

## Sample Questions

1. What symptom indicates data skew?  
2. How does AQE handle skewed joins?  
3. When is manual salting preferred over AQE?  
4. What is the trade-off of bucketing?  
5. How do you decide broadcast vs shuffle join?

## Answers

1. Straggler tasks with significantly higher durations and shuffle reads.  
2. Splits large partitions into smaller chunks dynamically during execution.  
3. For aggregations or when AQE thresholds don't trigger.  
4. Eliminates shuffle but requires table rewrites and stable keys.  
5. Broadcast when dimension < `autoBroadcastJoinThreshold`; shuffle for large-large joins.

## References

- [Adaptive Query Execution](https://spark.apache.org/docs/latest/sql-performance-tuning.html#adaptive-query-execution)
- [Bucketing guide](https://docs.databricks.com/optimizations/bucketing.html)

---

Previous: [Complex Data Types](./03-complex-data-types.md)
