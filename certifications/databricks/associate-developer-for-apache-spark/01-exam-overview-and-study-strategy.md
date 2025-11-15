# Exam Overview and Study Strategy

## Overview

Defines scope, weighting, and practical approach to mastering Spark APIs for the Associate Developer exam.

## Prerequisites

- Basic familiarity with Spark sessions (`spark`), DataFrame creation, and file IO.

## Concepts

- Exam domains: DataFrame operations, Spark SQL, joins, aggregations, performance basics, streaming fundamentals.
- Depth vs breadth: Prioritize fluency in API patterns over memorizing niche configs.
- Hands-on iteration: Small reproducible datasets to test transformations.

## Hands-on Walkthrough

### Create a Small Synthetic DataFrame (PySpark)

```python
from pyspark.sql import Row
rows = [Row(id=i, category='A' if i % 2==0 else 'B', value=float(i)) for i in range(10)]
df = spark.createDataFrame(rows)
df.show()
```

### Quick Exploration Tasks

```python
df.printSchema()
df.groupBy('category').avg('value').show()
df.filter("value > 5").select('id','value').show()
```

## Study Strategy

- Daily drills: Write 5 transformations (filter, select, withColumn, groupBy, join).
- Build cheat sheets: Common functions (string, date, aggregation).
- Optimize: Compare runtime of cached vs non-cached aggregations.
- Streaming mini-lab: File stream → aggregate → console sink.

## Production Considerations

- Reproducibility: Parameterize paths; avoid hard-coded cluster-only configs.
- Data contracts: Validate schema early; fail fast on mismatch.
- Cost awareness: Cache only if reused; unpersist aggressively.

## Troubleshooting

- Silent errors: Use `explain()` for unresolved logical plans.
- Wide scans: Limit columns early to reduce I/O.
- Ambiguous column references: Disambiguate with aliases post-join.

## Sample Questions

1. What API area carries most weight in the exam?  
2. Why is `explain()` useful during preparation?  
3. When should you cache a DataFrame?  
4. Why prefer synthetic data during drills?  
5. What principle guides join column handling?

## Answers

1. DataFrame transformations and aggregations.  
2. Reveals logical/physical plan for understanding execution.  
3. When reused multiple times in expensive downstream operations.  
4. Fast iteration and deterministic scenarios for debugging.  
5. Explicit aliasing to avoid ambiguous column names.

## References

- [Spark Programming Guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html)
- [DataFrame Operations](https://spark.apache.org/docs/latest/sql-programming-guide.html)

---

Next: [Spark Architecture and Execution Model](./02-spark-architecture-and-execution-model.md)
