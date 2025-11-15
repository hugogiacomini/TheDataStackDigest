# Combine DataFrames: Joins

## Overview

Perform inner, left, broadcast, multi-key, cross joins, and unions.

## Join Types

```python
# Inner join
inner = orders.join(customers, orders.customer_id == customers.customer_id, "inner")

# Left join
left = orders.join(customers, "customer_id", "left")

# Broadcast join hint (small dimension)
from pyspark.sql.functions import broadcast
broadcast_j = orders.join(broadcast(customers), "customer_id")

# Multi-key join
multi_key = df1.join(df2, (df1.key1 == df2.key1) & (df1.key2 == df2.key2))

# Cross join (Cartesian product)
cross = df1.crossJoin(df2)
```

## Union Operations

```python
# Union (remove duplicates)
union = df1.union(df2).distinct()

# Union all (keep all rows including duplicates)
union_all = df1.union(df2)

# Union by name (align columns by name, not position)
union_named = df1.unionByName(df2)
```

## Best Practices

- Broadcast small (<10MB) lookup tables to avoid shuffle.  
- Use column name string for single-key join; explicit condition for complex.  
- Avoid cross join unless necessary; can explode row count.

## Sample Questions

1. When use broadcast join?  
2. Difference between `union` and `unionAll` in PySpark 3+?  
3. Join on multiple keys syntax?  
4. What risk does cross join pose?  
5. Why `unionByName` over `union`?

## Answers

1. When one side is small enough to fit in executor memory.  
2. Both behave like SQL UNION ALL (no dedup); use `.distinct()` for dedup.  
3. `df1.join(df2, (df1.k1 == df2.k1) & (df1.k2 == df2.k2))`.  
4. Cartesian explosion leading to massive result set.  
5. Aligns by column name tolerating different ordering.

## References

- [Joins](https://spark.apache.org/docs/latest/sql-programming-guide.html#joins)

---

Previous: [Date Manipulation](./04-date-manipulation.md)  
Next: [Input/Output Operations](./06-input-output-operations.md)
