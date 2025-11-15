# Databricks Certified Associate Developer for Apache Spark – Study Guide

## Overview

Exam-aligned study guide organized by official outline sections. Each bullet from the outline maps to a dedicated file with: Overview, Concepts, Hands-on Examples, Best Practices, Sample Questions, Answers, References, and Prev/Next navigation.

## Audience

Data engineers with basic Spark experience seeking certification-level proficiency (Python or Scala), focusing on DataFrame API, Spark SQL, performance fundamentals, Structured Streaming, Spark Connect, and Pandas API on Spark.

## Prerequisites

- Familiarity with Spark sessions and DataFrame API
- Basic understanding of cluster concepts (driver, executor, partitions)
- Experience reading/writing Parquet or Delta tables
- Python (PySpark) fundamentals (or Scala equivalents)

## Section Index

### 01. Apache Spark Architecture and Components

- 01 Advantages & Challenges: [01-advantages-and-challenges-implementing-spark](./01-apache-spark-architecture-and-components/01-advantages-and-challenges-implementing-spark.md)
- 02 Core Components & Cluster Architecture: [02-core-components-cluster-architecture](./01-apache-spark-architecture-and-components/02-core-components-cluster-architecture.md)
- 03 Architecture: DataFrames, Session, Caching, Storage, GC: [03-architecture-dataframes-session-caching-storage-gc](./01-apache-spark-architecture-and-components/03-architecture-dataframes-session-caching-storage-gc.md)
- 04 Execution Hierarchy: [04-execution-hierarchy](./01-apache-spark-architecture-and-components/04-execution-hierarchy.md)
- 05 Partitioning & Shuffles Configuration: [05-partitioning-shuffles-configuration](./01-apache-spark-architecture-and-components/05-partitioning-shuffles-configuration.md)
- 06 Actions, Transformations, Lazy Evaluation: [06-actions-transformations-lazy-evaluation](./01-apache-spark-architecture-and-components/06-actions-transformations-lazy-evaluation.md)
- 07 Spark Module Features: [07-spark-module-features](./01-apache-spark-architecture-and-components/07-spark-module-features.md)

### 02. Using Spark SQL

- 01 Data Sources Read/Write Partitioning: [01-data-sources-read-write-partitioning](./02-using-spark-sql/01-data-sources-read-write-partitioning.md)
- 02 SQL Queries on Files & Save Modes: [02-sql-queries-on-files-save-modes](./02-using-spark-sql/02-sql-queries-on-files-save-modes.md)
- 03 Persistent Tables, Sorting, Partitioning: [03-persistent-tables-sorting-partitioning](./02-using-spark-sql/03-persistent-tables-sorting-partitioning.md)
- 04 Register Temp Views: [04-register-temp-views](./02-using-spark-sql/04-register-temp-views.md)

### 03. Developing DataFrame/Dataset API Applications

- 01 Column & Row Manipulation: [01-column-row-manipulation](./03-developing-dataframe-dataset-api-applications/01-column-row-manipulation.md)
- 02 Deduplication & Validation: [02-deduplication-validation](./03-developing-dataframe-dataset-api-applications/02-deduplication-validation.md)
- 03 Aggregate Operations: [03-aggregate-operations](./03-developing-dataframe-dataset-api-applications/03-aggregate-operations.md)
- 04 Date Manipulation: [04-date-manipulation](./03-developing-dataframe-dataset-api-applications/04-date-manipulation.md)
- 05 Combine DataFrames (Joins): [05-combine-dataframes-joins](./03-developing-dataframe-dataset-api-applications/05-combine-dataframes-joins.md)
- 06 Input/Output Operations & Schemas: [06-input-output-operations](./03-developing-dataframe-dataset-api-applications/06-input-output-operations.md)
- 07 DataFrame Ops (Sort, Iterate, Schema, Convert): [07-dataframe-operations](./03-developing-dataframe-dataset-api-applications/07-dataframe-operations.md)
- 08 User-Defined Functions: [08-user-defined-functions](./03-developing-dataframe-dataset-api-applications/08-user-defined-functions.md)
- 09 Broadcast Variables & Accumulators: [09-broadcast-accumulators](./03-developing-dataframe-dataset-api-applications/09-broadcast-accumulators.md)
- 10 Broadcast Joins: [10-broadcast-joins](./03-developing-dataframe-dataset-api-applications/10-broadcast-joins.md)

### 04. Troubleshooting and Tuning

- 01 Performance Tuning Strategies: [01-performance-tuning-strategies](./04-troubleshooting-and-tuning/01-performance-tuning-strategies.md)
- 02 Adaptive Query Execution (AQE): [02-adaptive-query-execution](./04-troubleshooting-and-tuning/02-adaptive-query-execution.md)
- 03 Logging & Monitoring (Driver/Executor/OOM/Underutilization): [03-logging-monitoring](./04-troubleshooting-and-tuning/03-logging-monitoring.md)

### 05. Structured Streaming

- 01 Streaming Engine Explanation: [01-streaming-engine-explanation](./05-structured-streaming/01-streaming-engine-explanation.md)
- 02 Create & Write Streaming DataFrames: [02-create-write-streaming-dataframes](./05-structured-streaming/02-create-write-streaming-dataframes.md)
- 03 Streaming Operations (Selection/Window/Aggregation): [03-streaming-operations](./05-structured-streaming/03-streaming-operations.md)
- 04 Streaming Deduplication (With/Without Watermark): [04-streaming-deduplication](./05-structured-streaming/04-streaming-deduplication.md)

### 06. Spark Connect

- 01 Spark Connect Features: [01-spark-connect-features](./06-spark-connect/01-spark-connect-features.md)
- 02 Deployment Modes (Client/Cluster/Local): [02-deployment-modes](./06-spark-connect/02-deployment-modes.md)

### 07. Using Pandas API on Spark

- 01 Pandas API Advantages: [01-pandas-api-advantages](./07-using-pandas-api-on-spark/01-pandas-api-advantages.md)
- 02 Create & Invoke Pandas UDF: [02-pandas-udf](./07-using-pandas-api-on-spark/02-pandas-udf.md)

## Study Approach

1. Progress sequentially; do not skip performance & streaming sections.  
2. Reproduce examples; vary data volume & partition counts to observe plan changes.  
3. Use `explain()` frequently—look for `BroadcastHashJoin`, `AdaptiveSparkPlan`.  
4. Build a mini project: batch ingestion → transformations → streaming sink → optimization.  
5. Drill sample questions; create variants changing formats, join types, and output modes.

## References

- [Apache Spark Docs](https://spark.apache.org/docs/latest/)
- [Databricks Docs](https://docs.databricks.com/)
- [Delta Lake Docs](https://docs.delta.io/)
- [Pandas API on Spark](https://spark.apache.org/docs/latest/api/python/user_guide/pandas_on_spark/index.html)

---

Start: [Advantages & Challenges Implementing Spark](./01-apache-spark-architecture-and-components/01-advantages-and-challenges-implementing-spark.md)
