# Joins, Aggregations, and Window Functions

## Overview

Perform efficient joins, group aggregations, and window analytic functions critical for exam scenarios.

## Prerequisites

- DataFrame basics; understanding of grouping semantics.

## Concepts

- Join types: inner, left, right, full, semi, anti.
- Join strategies: broadcast vs shuffle; hints (`broadcast(df)`).
- Aggregations: `groupBy`, `agg`, multi-aggregations, distinct counts.
- Window functions: `partitionBy`, `orderBy`, frame clauses.

## Hands-on Walkthrough

### Standard Join

```python
users = spark.read.parquet("/mnt/data/users/")
orders = spark.read.parquet("/mnt/data/orders/")
joined = orders.join(users, orders.user_id == users.user_id, "inner")
```

### Broadcast Join Hint

```python
from pyspark.sql.functions import broadcast
joined_opt = orders.join(broadcast(users), "user_id")
```

### Aggregation with Multiple Functions

```python
sales = orders.groupBy("user_id").agg(
    F.count("*").alias("orders"),
    F.sum("amount").alias("total_amount"),
    F.avg("amount").alias("avg_amount")
)
```

### Window Ranking

```python
from pyspark.sql.window import Window
win = Window.partitionBy("user_id").orderBy(F.col("order_ts"))
ranked = orders.withColumn("order_rank", F.row_number().over(win))
```

### Rolling Window Example

```python
rolling_win = Window.partitionBy("user_id").orderBy("order_ts").rowsBetween(-3, 0)
orders_roll = orders.withColumn("rolling_sum", F.sum("amount").over(rolling_win))
```

## Production Considerations

- Broadcast when smaller side < threshold; confirm with `explain()`.
- Avoid `distinct` inside window definitions unnecessarily.
- Frame specification: Use bounded frames to control performance.

## Troubleshooting

- Skew in join keys: Repartition or salt key column.
- Memory pressure broadcasting: Ensure small DataFrame truly small.
- Unexpected duplicate rows: Validate join condition equality vs subset of columns.

## Sample Questions

1. When is a broadcast join preferred?  
2. What does a window partition define?  
3. How do you compute multiple aggregations efficiently?  
4. Which join type returns only matching left side rows?  
5. Why use frame clauses in window functions?

## Answers

1. When one side is small enough to send to all executors avoiding shuffle.  
2. Logical grouping on which window function resets ordering.  
3. Use `groupBy().agg()` with multiple expressions in one pass.  
4. Inner join.  
5. To bound the computation scope and control resource usage/performance.

## References

- [Joins](https://spark.apache.org/docs/latest/sql-programming-guide.html#joins)
- [Window Functions](https://spark.apache.org/docs/latest/sql-programming-guide.html#window-functions)

---

Previous: [Schema Handling and Data Sources](./05-schema-handling-and-data-sources.md)  
Next: [Performance Optimization: Partitioning, Caching, Broadcast](./07-performance-optimization-caching-partitioning-broadcast.md)
