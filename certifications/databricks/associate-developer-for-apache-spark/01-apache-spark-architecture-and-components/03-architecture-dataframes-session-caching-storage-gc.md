# Architecture: DataFrames, SparkSession, Caching, Storage Levels, GC

## Overview

Describe Spark application lifecycle, SparkSession role, DataFrame/Dataset abstraction, caching/storage levels, and garbage collection context.

## SparkSession Lifecycle

- Created once per application/notebook: `spark = SparkSession.builder.getOrCreate()`.
- Manages catalog interaction, configuration, and SQL context access.
- Reconfiguration (e.g., shuffle partitions) can occur during runtime via `spark.conf.set`.

## DataFrame & Dataset Concepts

- DataFrame: Distributed collection of rows with named columns; optimized by Catalyst.  
- Dataset (Scala/Java): Type-safe API atop DataFrame/Encoder layer.  
- Logical Plan → Optimized Plan → Physical Plan via Catalyst and Tungsten.

## Caching & Storage Levels

| Level | Description | Use Case |
|-------|-------------|----------|
| MEMORY_ONLY | Store deserialized objects in memory | Fast reuse, small dataset |
| MEMORY_AND_DISK | Spill to disk if not fitting memory | Larger reused datasets |
| DISK_ONLY | Persist on disk only | Rare; large sequential reuse |
| MEMORY_ONLY_SER | Serialized in memory | Reduces memory footprint at cost of CPU |

## When to Cache

- Expensive transformation reused multiple times.  
- Post-filter narrow dataset shared across forks.  
- Avoid caching ephemeral or single-use intermediates.

## Garbage Collection Considerations

- Large object churn from wide transformations can trigger frequent GC.  
- Mitigation: Prune columns early, avoid unnecessary repartitioning, unpersist promptly.

## Example

```python
spark.conf.set("spark.sql.shuffle.partitions", "32")
base = spark.read.parquet("/mnt/data/facts/")
filtered = base.filter("event_date >= '2025-11-01'").select("id","event_date","amount")
filtered.cache().count()  # materialize cache
```

## Sample Questions

1. What optimization framework rewrites logical plans?  
2. When choose MEMORY_AND_DISK over MEMORY_ONLY?  
3. Why unpersist cached DataFrames?  
4. How does Dataset differ from DataFrame in Scala?  
5. What triggers GC pressure in Spark workloads?

## Answers

1. Catalyst.  
2. When dataset does not fit fully in memory.  
3. Free up storage memory region and prevent eviction churn.  
4. Dataset adds type safety via Encoders.  
5. Large shuffle stages generating many temporary objects.

## References

- [Caching & Persistence](https://spark.apache.org/docs/latest/sql-programming-guide.html#caching-and-persistence)
- [Storage Levels](https://spark.apache.org/docs/latest/rdd-programming-guide.html#rdd-persistence)

---

Previous: [Core Components](./02-core-components-cluster-architecture.md)  
Next: [Execution Hierarchy](./04-execution-hierarchy.md)
