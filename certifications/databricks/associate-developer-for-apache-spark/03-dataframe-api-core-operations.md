# DataFrame API Core Operations

## Overview

Master column expressions, selection, filtering, grouping, ordering, and basic aggregations with the DataFrame API.

## Prerequisites

- Spark session available; test data loaded or synthetic generation ability.

## Concepts

- Immutable DataFrames: Each transform returns a new plan node.
- Column expressions: Built with functions or SQL expressions.
- Projection pushdown: Selecting fewer columns reduces IO.
- Predicate pushdown: Filters applied early for efficiency (supported formats).

## Hands-on Walkthrough

### Creating and Selecting Columns

```python
from pyspark.sql import functions as F

df = spark.read.format("json").load("/mnt/data/events/")
selected = df.select("user_id", F.col("event_type"), F.col("ts").cast("timestamp"))
```

### Filtering and Complex Conditions

```python
filtered = selected.filter((F.col("event_type") == "click") & (F.hour("ts") < 12))
```

### Aggregations and Grouping

```python
agg = filtered.groupBy("user_id").agg(F.count("*").alias("clicks_morning"))
```

### Ordering and Limiting

```python
top = agg.orderBy(F.col("clicks_morning").desc()).limit(10)
```

### Chained Operations

```python
result = (spark.read.parquet("/mnt/data/sales/")
          .filter("amount > 0")
          .groupBy("region")
          .agg(F.sum("amount").alias("total"))
          .orderBy(F.col("total").desc()))
```

## Production Considerations

- Column pruning: Explicit `select` avoids unnecessary serialization.
- Filter first: Reduce rows earlier in pipeline.
- Function catalog: Prefer built-in functions for optimization potential.

## Troubleshooting

- Null explosion: Use `coalesce()` / `fillna()` for downstream stability.
- Ambiguous columns after join: Prefix or rename with `alias()`.
- Performance drop: Inspect physical plan for multiple scans; cache if reused.

## Sample Questions

1. What benefits does predicate pushdown provide?  
2. Why chain transformations fluidly?  
3. When is explicit projection essential?  
4. How to handle duplicate column names after a join?  
5. What is the impact of using UDF over built-in?

## Answers

1. Reduces data read from storage improving performance.  
2. Enhances readability and allows optimizer to reorder/generate a single plan.  
3. When source tables have many columns; improves IO and memory usage.  
4. Use aliases or `withColumnRenamed` to disambiguate.  
5. UDFs can block certain optimizations and may be slower.

## References

- [DataFrame API Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes)
- [Built-in Functions](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html)

---

Previous: [Spark Architecture and Execution Model](./02-spark-architecture-and-execution-model.md)  
Next: [Transformations, Actions, and Lazy Evaluation](./04-transformations-actions-and-lazy-evaluation.md)
