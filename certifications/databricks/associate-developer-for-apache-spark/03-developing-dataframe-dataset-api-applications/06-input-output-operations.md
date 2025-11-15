# Input/Output Operations with Schemas

## Overview

Write, overwrite, read DataFrames specifying explicit schemas.

## Write Operations

```python
# Write Parquet overwrite
df.write.mode("overwrite").parquet("/mnt/data/output/")

# Write Delta append
df.write.mode("append").format("delta").saveAsTable("prod.transactions")

# Write CSV with options
df.write.mode("overwrite").option("header", True).csv("/mnt/data/export.csv")
```

## Read with Explicit Schema

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType

schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("amount", DoubleType(), True)
])

df = spark.read.schema(schema).json("/mnt/data/input/")
```

## Schema Evolution

```python
# Delta merge schema on write
df.write.mode("append").option("mergeSchema", "true").format("delta").saveAsTable("prod.sales")
```

## Best Practices

- Always define schemas for production pipelines (stability).  
- Use Delta for ACID guarantees and schema evolution.  
- Avoid schema inference on large datasets (performance overhead).

## Sample Questions

1. Why specify explicit schema on read?  
2. Which write mode replaces data?  
3. How enable schema evolution in Delta?  
4. Risk of schema inference?  
5. Difference between saveAsTable and save?

## Answers

1. Ensures type consistency and avoids inference errors.  
2. `overwrite`.  
3. Use `mergeSchema` option on write.  
4. Potential type mismatches and performance cost.  
5. `saveAsTable` registers in catalog; `save` writes to path only.

## References

- [I/O Operations](https://spark.apache.org/docs/latest/sql-data-sources.html)

---

Previous: [Joins](./05-combine-dataframes-joins.md)  
Next: [DataFrame Operations: Sort, Iterate, Print Schema, Convert](./07-dataframe-operations.md)
