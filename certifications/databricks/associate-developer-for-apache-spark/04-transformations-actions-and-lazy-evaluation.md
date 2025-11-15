# Transformations, Actions, and Lazy Evaluation

## Overview

Clarify the distinction between transformations (lazy) and actions (trigger execution) and how Spark builds and optimizes DAGs.

## Prerequisites

- Basic DataFrame operations; ability to run `explain()`.

## Concepts

- Lazy evaluation: Spark defers execution until an action.
- Transformations: Narrow (map, filter) vs wide (groupBy, join).
- Actions: `count`, `collect`, `take`, `show`, `foreach`, writes.
- DAG optimization: Catalyst optimizer rewrites logical plan before execution.

## Hands-on Walkthrough

### Building a Plan Without Execution

```python
base = spark.read.parquet("/mnt/data/logs/")
plan = (base.filter("status = 200")
            .select("user_id", "endpoint")
            .groupBy("endpoint")
            .count())
# No execution yet
```

### Trigger Execution with Action

```python
plan.show(5)  # Executes
count_val = plan.count()
```

### Explain Plan

```python
plan.explain(True)
```

### Avoid Unnecessary Actions

```python
# BAD: multiple actions cause repeated computation
plan.count()
plan.collect()

# BETTER: cache if reused
plan_cached = plan.cache()
plan_cached.count()
plan_cached.collect()
plan_cached.unpersist()
```

## Production Considerations

- Minimize actions: Each triggers a job; consolidate required outputs.
- Cache strategically: High reuse + expensive compute; monitor storage.
- Avoid `collect()` for large datasets; use limits or aggregates instead.

## Troubleshooting

- Unexpected recompute: Missing cache or unpersisted intermediate results.
- Long-running job after `show()`: Large shuffle preceding action; inspect plan.
- Driver OOM: Excessive `collect()`; switch to `write` or sampled subsets.

## Sample Questions

1. What differentiates an action from a transformation?  
2. Why does Spark defer execution?  
3. When should caching be introduced?  
4. Why can repeated `count()` calls be inefficient?  
5. What risk does `collect()` introduce on large data?

## Answers

1. Actions trigger job execution and return a result; transformations build the logical plan.  
2. To optimize the full DAG prior to execution for performance.  
3. When an expensive result is reused multiple times.  
4. Each triggers a full job recomputation.  
5. Potential driver memory exhaustion and network overhead.

## References

- [Lazy Evaluation](https://spark.apache.org/docs/latest/sql-programming-guide.html#caching-and-persistence)
- [Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

---

Previous: [DataFrame API Core Operations](./03-dataframe-api-core-operations.md)  
Next: [Schema Handling and Data Sources](./05-schema-handling-and-data-sources.md)
