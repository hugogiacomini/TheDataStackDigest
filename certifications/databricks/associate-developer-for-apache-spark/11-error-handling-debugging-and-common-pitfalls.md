# Error Handling, Debugging, and Common Pitfalls

## Overview

Recognize and mitigate frequent Spark issues: schema mismatches, null handling, skew, driver memory blow-ups, and inefficient wide transformations.

## Prerequisites

- Experience running multiple notebook cells and reading stack traces.

## Concepts

- Schema mismatch: Write/append failures due to column order/type differences.
- Null handling: Defensive operations for aggregations and UDF inputs.
- Skew: Imbalanced partition workload causing stragglers.
- Driver overload: `collect()` misuse pulling large data locally.
- Repartition vs coalesce misuse: Over-shuffling or under-partitioning.

## Hands-on Walkthrough

### Detecting Skew (PySpark)

```python
from pyspark.sql import functions as F

counts = df.groupBy("key").count()
max_count = counts.agg(F.max("count")).first()[0]
avg_count = counts.agg(F.avg("count")).first()[0]
ratio = max_count / avg_count
print("Skew ratio", ratio)
```

### Safe Aggregation

```python
agg = df.groupBy("category").agg(F.sum(F.coalesce("amount", F.lit(0))).alias("amt"))
```

### Limiting Data for Debug

```python
sample_debug = df.limit(1000).toPandas()  # safe small sample
```

### Avoiding Driver OOM

```python
# BAD
big = df.collect()  # may explode driver
# BETTER
subset = df.select("important_col").limit(5000).toPandas()
```

## Production Considerations

- Validate schema upfront; enforce with ingestion contract tests.
- Monitor skew metrics; apply salting or repartitioning.
- Replace broad `select *` with explicit columns to reduce memory.

## Troubleshooting

- Unexpected null results: Check implicit casts, missing columns after joins.
- Long tail tasks: Partition skew; inspect with Spark UI stages.
- Frequent recomputation: Add caching where reused; verify lineage.

## Sample Questions

1. What causes straggler tasks?  
2. Safe alternative to `collect()` for inspection?  
3. How handle nulls in summations?  
4. Indicator of skew severity?  
5. When use `repartition` instead of `coalesce`?

## Answers

1. Skewed partitions with disproportionate data volume.  
2. `limit().toPandas()` on a small subset.  
3. Wrap column with `coalesce(col, lit(0))` before `sum`.  
4. Max vs average partition row count ratio.  
5. When increasing number of partitions or rebalancing for parallelism.

## References

- [Debugging Spark](https://spark.apache.org/docs/latest/troubleshooting.html)
- [Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

---

Previous: [Structured Streaming Basics](./10-structured-streaming-basics.md)  
Next: [Practice Exam Questions](./12-practice-exam-questions.md)
