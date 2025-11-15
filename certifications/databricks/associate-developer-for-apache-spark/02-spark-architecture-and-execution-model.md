# Spark Architecture and Execution Model

## Overview

Understand driver vs executors, tasks, stages, shuffles, and physical vs logical plans to reason about performance and correctness.

## Prerequisites

- Basic DataFrame operations and Spark UI familiarity.

## Concepts

- Driver: Orchestrates job; builds logical plan → physical plan.
- Executors: Run tasks; hold cached blocks; ephemeral.
- Stages: Boundaries at shuffle points; DAG segmentation.
- Tasks: Unit of parallel work on partitions.
- Shuffle: Data redistribution for wide transformations (groupBy, join, distinct).

## Hands-on Walkthrough

### Inspect Execution Plan

```python
spark.conf.set("spark.sql.shuffle.partitions", "8")
df = spark.range(0, 100).withColumn("grp", (spark.range(0,100).id % 3))
df.groupBy("grp").count().explain(True)
```

### Force Shuffle vs Narrow Transformation

```python
narrow = df.filter("grp = 1")  # no shuffle
wide = df.groupBy("grp").count()  # shuffle boundary
```

### Spark UI Checks

1. Run an aggregation notebook cell.
2. Open Spark UI → SQL tab → View physical plan; note Exchange operators (shuffles).

## Production Considerations

- Partition sizing: Too many partitions → overhead; too few → underutilization.
- Broadcast join: Avoid large shuffles when one side small.
- Executor memory: Caching strategy depends on memory vs recompute cost.

## Troubleshooting

- Straggler tasks: Investigate skew; consider salting or repartitioning.
- OOM in executors: Reduce shuffle partitions or filter earlier.
- Excess stages: Combined operations produce unnecessary shuffles; check plan.

## Sample Questions

1. What triggers stage boundaries?  
2. Why monitor Exchange operators?  
3. What differentiates narrow vs wide transformations?  
4. How can skew affect execution?  
5. When to adjust `spark.sql.shuffle.partitions`?

## Answers

1. Shuffle operations (wide transformations).  
2. They identify costly data redistribution events.  
3. Narrow operate within partitions; wide require data movement.  
4. Causes uneven task durations and cluster inefficiency.  
5. To balance parallelism and overhead based on dataset size.

## References

- [Spark Architecture Concepts](https://spark.apache.org/docs/latest/cluster-overview.html)
- [SQL Execution Plans](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

---

Previous: [Exam Overview and Study Strategy](./01-exam-overview-and-study-strategy.md)  
Next: [DataFrame API Core Operations](./03-dataframe-api-core-operations.md)
