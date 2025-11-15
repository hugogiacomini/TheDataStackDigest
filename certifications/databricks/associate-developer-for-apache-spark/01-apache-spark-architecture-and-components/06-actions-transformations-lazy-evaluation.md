# Actions, Transformations, and Lazy Evaluation

## Overview

Differentiate between transformations (lazy plan building) and actions (execution triggers) with implications for optimization.

## Transformations

- Lazy: `filter`, `select`, `withColumn`, `groupBy`, `join`.  
- Narrow vs wide: wide introduces shuffle & stage boundary.

## Actions

- Trigger jobs: `count`, `collect`, `show`, `take`, writing (`write.format(...).save`).
- Each action = new job; consolidate where possible.

## Lazy Evaluation Benefits

- Plan optimization occurs before execution (predicate & projection pushdown).  
- Avoid unnecessary intermediate materialization.

## Example

```python
base = spark.read.parquet("/mnt/data/events/")
logic = base.filter("severity = 'WARN'").groupBy("service").count()
logic.explain(True)  # still lazy
logic.show()  # executes
```

## Best Practices

- Use `explain()` to inspect plan before heavy actions.  
- Avoid repeated actions on same lineage; cache if reused.  
- Replace `collect()` with sampled/limited inspection.

## Sample Questions

1. Why is lazy evaluation important?  
2. Give two examples of actions.  
3. What transformation type triggers shuffle?  
4. How minimize duplicate execution?  
5. Why prefer `show()` over `collect()` for large data?

## Answers

1. Enables global optimization and prevents unnecessary computation.  
2. `count`, `show` (many others).  
3. Wide transformations (e.g., groupBy).  
4. Cache or persist intermediate reused DataFrames.  
5. Limits displayed rows avoiding driver memory pressure.

## References

- [Lazy Evaluation Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)

---

Previous: [Partitioning & Shuffles](./05-partitioning-shuffles-configuration.md)  
Next: [Spark Module Features](./07-spark-module-features.md)
