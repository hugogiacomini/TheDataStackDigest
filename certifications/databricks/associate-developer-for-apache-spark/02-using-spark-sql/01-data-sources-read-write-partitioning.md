# Data Sources: Read, Write, Overwrite, Partitioning

## Overview

Efficiently read from and write to DataFrames using common sources (JDBC, files) with overwrite modes and column partitioning.

## Common Data Sources

- JDBC: Relational databases (Postgres, MySQL, SQL Server).  
- Files: Parquet, JSON, CSV, ORC, Delta, Avro.  
- Cloud Storage: S3, Azure Blob/ADLS, GCS.

## Read Patterns

```python
# JDBC
jdbc_df = (spark.read.format("jdbc")
           .option("url", "jdbc:postgresql://host:5432/db")
           .option("dbtable", "sales")
           .option("user", "admin")
           .option("password", "secret")
           .load())

# Parquet with partitioning
parquet_df = spark.read.parquet("/mnt/data/sales/")

# CSV with schema
from pyspark.sql.types import StructType, StructField, StringType, DoubleType
schema = StructType([
    StructField("id", StringType(), False),
    StructField("amount", DoubleType(), True)
])
csv_df = spark.read.schema(schema).csv("/mnt/data/input.csv", header=True)
```

## Write Patterns

```python
# Overwrite entire table
df.write.mode("overwrite").parquet("/mnt/data/output/")

# Append new data
df.write.mode("append").format("delta").saveAsTable("prod.sales.transactions")

# Partition by column
df.write.mode("overwrite").partitionBy("date", "region").parquet("/mnt/data/sales/")
```

## SaveModes

| Mode | Behavior |
|------|----------|
| overwrite | Replace all existing data |
| append | Add new data to existing |
| error / errorifexists | Fail if target exists (default) |
| ignore | Silent no-op if target exists |

## Best Practices

- Partition by high-selectivity columns (date, region) avoiding excessive cardinality.  
- Use Delta for ACID semantics and schema evolution.  
- Batch JDBC reads with partitionColumn/numPartitions for parallelism.

## Sample Questions

1. Which write mode replaces existing data?  
2. When use `partitionBy`?  
3. Why prefer Delta over Parquet for updates?  
4. How parallelize JDBC reads?  
5. What is default write mode?

## Answers

1. `overwrite`.  
2. To optimize query pruning on frequently filtered columns.  
3. Delta supports ACID and efficient updates/deletes.  
4. Use `partitionColumn`, `lowerBound`, `upperBound`, `numPartitions`.  
5. `error` (fails if target exists).

## References

- [Data Sources](https://spark.apache.org/docs/latest/sql-data-sources.html)
- [JDBC Guide](https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html)

---

Next: [Execute SQL on Files Directly](./02-sql-queries-on-files-save-modes.md)
