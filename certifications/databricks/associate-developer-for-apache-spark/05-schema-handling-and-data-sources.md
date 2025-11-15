# Schema Handling and Data Sources

## Overview

Work with explicit vs inferred schemas, common file formats, partition discovery, and options affecting read/write correctness.

## Prerequisites

- Ability to read JSON/Parquet; basic understanding of schema evolution.

## Concepts

- Inference: Convenience with JSON/CSV; risk of inconsistent types.
- Explicit schema: Stable pipelines; defined via `StructType`.
- Common formats: Parquet (columnar), JSON (semi-structured), CSV (text), Delta (ACID).
- Partition discovery: Directory-based partition columns for pruning.

## Hands-on Walkthrough

### Explicit Schema (PySpark)

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("category", StringType(), True),
    StructField("amount", DoubleType(), True)
])

transactions = spark.read.schema(schema).json("/mnt/raw/transactions/")
```

### Schema Inference

```python
auto = spark.read.option("inferSchema", True).csv("/mnt/raw/simple.csv", header=True)
```

### Partitioned Parquet Read

Folder layout: `/mnt/data/sales/date=2025-11-01/region=EMEA/*.parquet`

```python
sales = spark.read.parquet("/mnt/data/sales/")
sales.printSchema()  # includes date, region as inferred partitions
filtered = sales.filter("region = 'EMEA' AND date >= '2025-11-01'")
```

### Delta Write with Overwrite Schema

```python
transactions.write.format("delta").mode("overwrite").option("overwriteSchema", "true").saveAsTable("prod.sales.transactions_delta")
```

## Production Considerations

- Always define schemas for critical pipelines (stability + evolution control).
- Avoid `inferSchema` on large multi-file reads; can cause performance overhead.
- Partition strategy: High-cardinality caution; balance pruning vs small files.

## Troubleshooting

- Column mismatch writes: Use `mergeSchema` or align columns before appending.
- Inference mistakes (string instead of int): Clean sample data or enforce schema.
- Small file problem: Repartition before write to manage file counts.

## Sample Questions

1. Why prefer explicit schemas for ingestion?  
2. What benefit does Parquet offer over CSV?  
3. How does partition discovery assist performance?  
4. When use `overwriteSchema`?  
5. Risk of relying on schema inference?

## Answers

1. Ensures consistent types and guards against drift.  
2. Columnar storage enabling predicate/column pruning.  
3. Enables partition pruning reducing scanned data.  
4. When evolving schema intentionally during overwrite operations.  
5. Potential mis-typing causing downstream errors and inconsistent logic.

## References

- [Data Sources](https://spark.apache.org/docs/latest/sql-data-sources.html)
- [Delta Lake Schema Evolution](https://docs.delta.io/latest/delta-schema-evolution.html)

---

Previous: [Transformations, Actions, and Lazy Evaluation](./04-transformations-actions-and-lazy-evaluation.md)  
Next: [Joins, Aggregations, and Window Functions](./06-joins-aggregations-and-window-functions.md)
