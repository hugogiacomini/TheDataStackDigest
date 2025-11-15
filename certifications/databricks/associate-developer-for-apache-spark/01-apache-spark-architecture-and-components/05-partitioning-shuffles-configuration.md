# Partitioning, Shuffles, and Configuration

## Overview

Configure partition counts, understand shuffle mechanics, and tune related parameters for performance.

## Partition Fundamentals

- Each partition feeds a task; number influences parallelism.  
- Default shuffle partitions: `spark.sql.shuffle.partitions` (often 200 in vanilla Spark).  
- Coalesce reduces partitions without full shuffle; repartition triggers shuffle.

## Shuffle Mechanics

- Occurs during wide transformations (join, groupBy, distinct, orderBy).  
- Data is hashed/sorted and redistributed to target partitions.  
- Spill occurs when shuffle data exceeds memory; written to disk.

## Configuration Examples

```python
spark.conf.set("spark.sql.shuffle.partitions", "64")
# Optimize for medium dataset size
```

```python
large = spark.read.parquet("/mnt/data/large/")
balanced = large.repartition(128, "region")  # hash partition by column
reduced = balanced.coalesce(32)  # reduce for final write
```

## Best Practices

- Avoid tiny partitions (<128MB per partition) for large datasets.  
- Partition columns should have adequate cardinality; avoid exploding small files.  
- Repartition before wide aggregations for even load.

## Sample Questions

1. What parameter controls shuffle output partitions?  
2. Difference between `coalesce` and `repartition`?  
3. What triggers a shuffle?  
4. Why repartition on join key?  
5. Risk of excessive small partitions?

## Answers

1. `spark.sql.shuffle.partitions`.  
2. Coalesce merges partitions without full shuffle; repartition redistributes via shuffle.  
3. Wide transformations requiring data redistribution.  
4. Improve balance and reduce skew across tasks.  
5. Increased scheduling overhead and metadata pressure.

## References

- [Partition Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

---

Previous: [Execution Hierarchy](./04-execution-hierarchy.md)  
Next: [Actions, Transformations, and Lazy Evaluation](./06-actions-transformations-lazy-evaluation.md)
