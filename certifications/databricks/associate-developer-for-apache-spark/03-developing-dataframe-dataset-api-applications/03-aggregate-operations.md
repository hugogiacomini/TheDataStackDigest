# Aggregate Operations

## Overview

Perform count, approx_count_distinct, mean, summary statistics on DataFrames.

## Common Aggregations

```python
from pyspark.sql import functions as F

# Simple aggregations
stats = df.groupBy("region").agg(
    F.count("*").alias("total_orders"),
    F.sum("amount").alias("revenue"),
    F.avg("amount").alias("avg_amount"),
    F.min("amount").alias("min"),
    F.max("amount").alias("max")
)

# Approximate distinct count (faster for large datasets)
approx_users = df.agg(F.approx_count_distinct("user_id", rsd=0.05)).collect()[0][0]

# Summary statistics
df.select("amount").summary("count", "mean", "stddev", "min", "25%", "50%", "75%", "max").show()

# Multiple groupings
multi = df.groupBy("region", "product_category").agg(F.sum("amount"))
```

## Best Practices

- Use `approx_count_distinct` for high-cardinality columns; much faster than exact.  
- Consolidate multiple aggs in one `agg()` call for efficiency.  
- Prefer built-in aggregation functions over UDFs.

## Sample Questions

1. Difference between `count` and `approx_count_distinct`?  
2. How compute multiple aggregates in one pass?  
3. What does `summary()` provide?  
4. When use `approx_count_distinct`?  
5. Why group by multiple columns?

## Answers

1. `count` is exact row count; `approx_count_distinct` estimates unique values.  
2. Use `groupBy().agg(func1, func2, ...)`.  
3. Statistical summary (count, mean, std, percentiles).  
4. When exact distinct count is too expensive on large data.  
5. To compute metrics at finer granularity (e.g., per region and category).

## References

- [Aggregation Functions](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html#aggregate-functions)

---

Previous: [Deduplication & Validation](./02-deduplication-validation.md)  
Next: [Date Data Type Manipulation](./04-date-manipulation.md)
