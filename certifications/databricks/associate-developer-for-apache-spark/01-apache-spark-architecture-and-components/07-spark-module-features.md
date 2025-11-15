# Spark Module Features

## Overview

Identify capabilities of Core, Spark SQL, DataFrames, Pandas API on Spark, Structured Streaming, and MLlib.

## Modules

- Core: RDD API, scheduling, memory management, fault tolerance lineage.  
- Spark SQL / DataFrames: Declarative queries, Catalyst optimizations, unified data access.  
- Pandas API on Spark: Pandas-like syntax at scale; leverages Arrow & vectorization.  
- Structured Streaming: Incremental processing, event-time semantics, exactly-once sinks.  
- MLlib: Distributed algorithms (classification, regression, clustering, feature extraction).

## Choosing APIs

| Use Case | Recommended Module |
|----------|--------------------|
| Ad-hoc analytics | Spark SQL / DataFrames |
| Complex transformations with Python idioms | Pandas API on Spark |
| Real-time ingestion & aggregates | Structured Streaming |
| ML pipeline training | MLlib + DataFrames |

## Example: Mixed Usage

```python
# DataFrames + SQL
spark.sql("SELECT COUNT(*) FROM prod.sales").show()

# Pandas API on Spark
import pyspark.pandas as ps
psdf = ps.read_parquet("/mnt/data/sales/")
psdf.groupby("region")["amount"].mean()
```

## Sample Questions

1. What advantage does Spark SQL have over RDD operations?  
2. When prefer Pandas API on Spark?  
3. Key feature of Structured Streaming?  
4. Role of MLlib?  
5. Why DataFrames over RDD for optimization?

## Answers

1. Catalyst optimizations and concise declarative syntax.  
2. When leveraging familiar Pandas operations that scale out.  
3. Provides incremental micro-batch processing with fault tolerance.  
4. Distributed machine learning algorithms & feature utilities.  
5. Enables rule-based and cost-based optimizations not available to raw RDDs.

## References

- [Spark SQL](https://spark.apache.org/sql/)
- [Pandas API on Spark](https://spark.apache.org/docs/latest/api/python/user_guide/pandas_on_spark/index.html)
- [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [MLlib Guide](https://spark.apache.org/docs/latest/ml-guide.html)

---

Previous: [Actions & Lazy Evaluation](./06-actions-transformations-lazy-evaluation.md)  
Next: [Using Spark SQL: Data Sources](../02-using-spark-sql/01-data-sources-read-write-partitioning.md)
