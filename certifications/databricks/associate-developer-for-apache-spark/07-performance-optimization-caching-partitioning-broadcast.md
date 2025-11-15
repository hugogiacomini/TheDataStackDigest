# Performance Optimization: Partitioning, Caching, Broadcast

## Overview

Key tactics for improving query execution: partitioning strategy, caching/persisting, broadcast joins, and controlling shuffle size.

## Prerequisites

- Familiarity with joins, aggregations, Spark UI.

## Concepts

- Partitioning: Influences parallelism and scan pruning.
- Caching vs persistence: Memory vs disk/storage levels.
- Broadcast: Distribute small lookup DataFrame to all executors.
- Shuffle partitions: Controlled by `spark.sql.shuffle.partitions`.

## Hands-on Walkthrough

### Adjust Shuffle Partitions

```python
spark.conf.set("spark.sql.shuffle.partitions", "64")
```

### Cache a Frequent Intermediate

```python
wide = spark.read.parquet("/mnt/data/facts/")\
        .join(spark.read.parquet("/mnt/data/dim_region/"), "region_id")\
        .filter("event_date >= '2025-11-01'")
wide_cached = wide.cache()
wide_cached.count()  # materialize
```

### Broadcast a Small Dimension

```python
from pyspark.sql.functions import broadcast
regions = spark.read.parquet("/mnt/data/dim_region/")
optimized = wide.join(broadcast(regions), "region_id")
```

### Persist at Different Storage Level

```python
from pyspark import StorageLevel
alt = wide.persist(StorageLevel.MEMORY_AND_DISK)
```

## Production Considerations

- Monitor storage usage after caching; unpersist when done.
- Avoid over-partitioning tiny datasets (task overhead).  
- Coalesce for final small output writes; repartition for balanced large shuffles.

## Troubleshooting

- Cache not reused: Ensure downstream references same DataFrame variable.
- Spill to disk: Consider reducing partition size or increasing executor memory.
- Broadcast failure: DataFrame too large; adjust threshold or different strategy.

## Sample Questions

1. When is caching beneficial?  
2. What does broadcasting achieve?  
3. Difference between `repartition` and `coalesce`?  
4. Why tune `spark.sql.shuffle.partitions`?  
5. Risk of over-caching?

## Answers

1. When an expensive intermediate is reused multiple times.  
2. Eliminates shuffle for the small side of a join.  
3. `repartition` adds a full shuffle; `coalesce` reduces partitions without full shuffle.  
4. Balances parallelism vs overhead relative to data size.  
5. Memory pressure leading to eviction/spill and degraded performance.

## References

- [Caching and Persistence](https://spark.apache.org/docs/latest/sql-programming-guide.html#caching-and-persistence)
- [Broadcast Joins](https://spark.apache.org/docs/latest/sql-performance-tuning.html#broadcast-hint)

---

Previous: [Joins, Aggregations, and Window Functions](./06-joins-aggregations-and-window-functions.md)  
Next: [User-Defined Functions and Built-ins](./08-user-defined-functions-and-builtins.md)
