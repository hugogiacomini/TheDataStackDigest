# Execution Hierarchy

## Overview

Explain Spark's hierarchy: Application → Job → Stage → Task and how transformations/actions map onto this structure.

## Hierarchy Breakdown

| Level | Description | Trigger |
|-------|-------------|--------|
| Application | User program with driver & executors | SparkSession creation |
| Job | Result of an action (e.g., `count`, `write`) | Action invoked |
| Stage | Set of tasks separated by shuffle boundaries | Wide transformation encountered |
| Task | Work unit on a partition | Assigned during stage scheduling |

## Wide vs Narrow Impact

- Narrow: Map/filter combine into single stage.  
- Wide: GroupBy/join introduces shuffle → new stage.

## Example

```python
orders = spark.read.parquet("/mnt/data/orders/")
agg = orders.groupBy("customer_id").count()  # wide -> shuffle -> stage boundary
agg.count()  # action launches a job containing stages
```

Use Spark UI → Jobs & Stages to visualize DAG separation.

## Best Practices

- Reduce unnecessary shuffles (avoid repeated wide transformations).  
- Consolidate actions to minimize job count.  
- Monitor long tail tasks for skew.

## Sample Questions

1. What creates a new job?  
2. What event divides stages?  
3. What maps directly to tasks?  
4. Why identify wide transformations early?  
5. How many jobs from two successive `count()` calls?

## Answers

1. Invoking an action.  
2. Shuffle boundary from wide transformation.  
3. Individual partitions processed in parallel.  
4. To plan optimization and resource usage.  
5. Two jobs (each action triggers a job).

## References

- [Monitoring Jobs and Stages](https://spark.apache.org/docs/latest/web-ui.html)

---

Previous: [Architecture: DataFrames & Caching](./03-architecture-dataframes-session-caching-storage-gc.md)  
Next: [Partitioning, Shuffles, and Configuration](./05-partitioning-shuffles-configuration.md)
