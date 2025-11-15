# Column, Row, and Table Structure Manipulation

## Overview

Add, drop, split, rename columns; filter rows; explode arrays.

## Common Operations

```python
from pyspark.sql import functions as F

# Add column
df_with_flag = df.withColumn("is_premium", F.when(F.col("amount") > 100, True).otherwise(False))

# Drop columns
df_slim = df.drop("temp_col", "audit_ts")

# Rename column
df_renamed = df.withColumnRenamed("old_name", "new_name")

# Split column
df_split = df.withColumn("first_name", F.split(F.col("full_name"), " ").getItem(0))

# Filter rows
df_filtered = df.filter((F.col("status") == "active") & (F.col("amount") > 0))

# Explode array
df_exploded = df.withColumn("item", F.explode(F.col("items_array")))
```

## Best Practices

- Use `select` with explicit columns instead of `drop` when narrowing significantly.  
- Chain `withColumn` carefully; consider intermediate variables for readability.  
- Explode arrays early if downstream logic requires row-per-item.

## Sample Questions

1. How add a computed column?  
2. What does `explode` accomplish?  
3. Why prefer `select` over repeated `drop`?  
4. How split a string column?  
5. Combine multiple filter conditions?

## Answers

1. `withColumn` with expression.  
2. Converts array into multiple rows (one per element).  
3. More concise and avoids unnecessary plan nodes.  
4. Use `F.split(col, delimiter).getItem(index)`.  
5. Use `&` (and), `|` (or) with parentheses for precedence.

## References

- [Column Functions](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html)

---

Next: [Data Deduplication and Validation](./02-deduplication-validation.md)
