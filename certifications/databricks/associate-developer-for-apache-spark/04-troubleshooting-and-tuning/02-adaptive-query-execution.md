# Adaptive Query Execution (AQE) and Benefits

## Overview

AQE dynamically optimizes query plans during execution based on runtime statistics.

## Key Features

1. **Dynamic Coalescing of Shuffle Partitions**: Reduces small partitions post-shuffle.  
2. **Dynamic Switching of Join Strategies**: Converts sort-merge to broadcast join if small enough.  
3. **Dynamic Skew Join Optimization**: Splits skewed partitions for parallel processing.

## Enabling AQE

```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

## Example

```python
# AQE automatically adjusts at runtime
df = spark.read.parquet("/mnt/large_data/")
result = df.groupBy("region").agg(F.sum("amount"))
result.explain()  # Check for "AdaptiveSparkPlan"
```

## Benefits

- Reduces shuffle overhead by coalescing small partitions.  
- Improves join performance via dynamic broadcast.  
- Handles skew automatically without manual salting.

## Best Practices

- Enable AQE by default in Spark 3.0+.  
- Monitor query plans to verify adaptive optimizations applied.  
- Combine with proper initial partitioning for best results.

## Sample Questions

1. What is AQE?  
2. Three main AQE optimizations?  
3. How enable AQE?  
4. When does AQE convert joins?  
5. Does AQE require code changes?

## Answers

1. Adaptive Query Execution optimizes plans at runtime.  
2. Coalesce partitions, switch join strategies, optimize skew.  
3. Set `spark.sql.adaptive.enabled` to `true`.  
4. When post-shuffle size falls below broadcast threshold.  
5. No; transparent optimization layer.

## References

- [AQE Documentation](https://spark.apache.org/docs/latest/sql-performance-tuning.html#adaptive-query-execution)

---

Previous: [Performance Tuning Strategies](./01-performance-tuning-strategies.md)  
Next: [Logging and Monitoring](./03-logging-monitoring.md)
