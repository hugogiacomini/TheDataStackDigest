# Core Components of Cluster Architecture

## Overview

Identify roles of cluster manager, driver, executors, cores, and memory in Spark.

## Components

- Cluster Manager: Allocates resources (Databricks, YARN, Kubernetes).  
- Driver Node: Runs main application; constructs logical & physical plans; coordinates tasks.  
- Executors: JVM processes executing tasks; hold cached data & shuffle blocks.  
- CPU Cores: Parallel task slots within executors.  
- Memory: Divided between execution (shuffle, aggregation) and storage (cache/persist).

## Memory Regions (Simplified)

| Region | Purpose | Common Issues |
|--------|---------|---------------|
| Execution | Sorts, shuffles, joins | OOM from large wide transformations |
| Storage | Cached DataFrames | Eviction due to oversized cache |

## Example: Viewing Resources

Use Spark UI → Executors tab: inspect active/existing executors, memory & task metrics.

## Best Practices

- Right-Size Executors: Fewer large executors vs many small; minimize overhead.
- Core Allocation: Keep one core free for overhead; avoid saturating driver.
- Memory Tuning: Prefer DataFrame operations that leverage Tungsten optimization.

## Sample Questions

1. What is the driver's primary responsibility?  
2. Why should executor cores be tuned?  
3. What causes executor memory eviction events?  
4. Where do shuffle blocks reside during a wide stage?  
5. Why avoid over-provisioning tiny executors?

## Answers

1. Constructing plans and coordinating task scheduling/execution.  
2. Balance parallelism vs context-switch overhead.  
3. Cache pressure or insufficient storage memory region.  
4. On executors and possibly external storage if spilling.  
5. Excess overhead and degraded throughput due to scheduling complexity.

## References

- [Cluster Mode Overview](https://spark.apache.org/docs/latest/cluster-overview.html)

---

Previous: [Advantages and Challenges](./01-advantages-and-challenges-implementing-spark.md)  
Next: [Architecture: DataFrames, SparkSession, Caching, Storage Levels, GC](./03-architecture-dataframes-session-caching-storage-gc.md)
