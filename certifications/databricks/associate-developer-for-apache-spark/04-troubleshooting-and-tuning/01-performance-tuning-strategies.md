# Performance Tuning Strategies

## Overview

Optimize Spark performance through partitioning, repartitioning, coalescing, data skew mitigation, and shuffle reduction.

## Partitioning Strategies

```python
# Repartition for parallelism increase (triggers shuffle)
df_repart = df.repartition(200, "customer_id")

# Coalesce for parallelism reduction (avoids shuffle)
df_coalesced = df.coalesce(10)

# Partition by column for partitioned writes
df.write.partitionBy("year", "month").parquet("/mnt/partitioned/")
```

## Data Skew Mitigation

```python
# Add salt for skewed keys
from pyspark.sql import functions as F
salted = df.withColumn("salted_key", F.concat(F.col("key"), F.lit("_"), F.rand() * 10))
result = salted.groupBy("salted_key").agg(F.sum("amount"))

# Filter skewed keys separately
skewed_keys = ["key1", "key2"]
skewed_df = df.filter(F.col("key").isin(skewed_keys))
normal_df = df.filter(~F.col("key").isin(skewed_keys))
```

## Reducing Shuffles

```python
# Broadcast small tables
from pyspark.sql.functions import broadcast
df.join(broadcast(small_dim), "dim_id")

# Use narrow transformations (filter, select) before wide (groupBy, join)
filtered = df.filter(F.col("active") == True)
result = filtered.groupBy("region").agg(F.sum("amount"))
```

## Best Practices

- Match partitions to cluster cores for optimal parallelism.  
- Coalesce before writing small outputs to reduce file count.  
- Salt skewed keys; consider adaptive execution for automatic handling.

## Sample Questions

1. Difference between `repartition` and `coalesce`?  
2. How mitigate data skew?  
3. When broadcast a table?  
4. Why reduce shuffles?  
5. Optimal partition count?

## Answers

1. `repartition` triggers full shuffle; `coalesce` merges without shuffle.  
2. Salt keys, filter skewed separately, or enable AQE.  
3. When small (<threshold) to avoid shuffle join.  
4. Shuffles are expensive (network I/O, disk spills).  
5. 2-4x total executor cores for balanced parallelism.

## References

- [Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

---

Previous: [Broadcast Joins](../03-developing-dataframe-dataset-api-applications/10-broadcast-joins.md)  
Next: [Adaptive Query Execution (AQE)](./02-adaptive-query-execution.md)
