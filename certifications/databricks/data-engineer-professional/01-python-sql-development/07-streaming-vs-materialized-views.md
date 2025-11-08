
# Streaming Tables vs. Materialized Views

## Concept Definition

In Databricks, both **Streaming Tables** and **Materialized Views** are managed Delta Lake tables that are updated automatically as the data they depend on changes. However, they are designed for different use cases and have different characteristics. [1]

- **Streaming Tables** are designed for append-only streaming workloads. They process data that has been added since the last pipeline update and are stateful.

- **Materialized Views** are designed for batch workloads and are used to pre-compute results of complex queries. They are automatically and incrementally updated, but they are not designed for continuous, low-latency streaming.

## Key Differences

| Feature | Streaming Tables | Materialized Views |
|---|---|---|
| **Use Case** | Append-only streaming | Batch processing, pre-computation |
| **Processing** | Incremental, append-only | Incremental, re-computation |
| **Latency** | Low | Higher |
| **State** | Stateful | Stateless (recomputed from source) |
| **Update Trigger** | Continuous or scheduled | Scheduled or on-demand |

## Python/PySpark Examples

### Creating a Streaming Table

```python
import dlt

@dlt.table
def my_streaming_table():
  return (
    spark.readStream.format("cloudFiles")
      .option("cloudFiles.format", "json")
      .load("/path/to/source/data")
  )
```

### Creating a Materialized View

```python
import dlt

@dlt.table
def my_materialized_view():
  return (
    dlt.read("my_streaming_table")
      .groupBy("column_a")
      .agg(sum("column_b").alias("total_b"))
  )
```

## SQL Examples

### Creating a Streaming Table

```sql
CREATE STREAMING LIVE TABLE my_streaming_table
AS SELECT * FROM cloud_files("/path/to/source/data", "json");
```

### Creating a Materialized View

```sql
CREATE MATERIALIZED VIEW my_materialized_view
AS SELECT
  column_a,
  SUM(column_b) as total_b
FROM LIVE.my_streaming_table
GROUP BY column_a;
```

## Best Practices

- **Use Streaming Tables for Ingestion**: Streaming tables are ideal for ingesting raw data from streaming sources like Kafka or from files in cloud storage.
- **Use Materialized Views for Aggregations**: Materialized views are well-suited for pre-computing aggregations and other complex transformations that are queried frequently.
- **Chain them together**: A common pattern is to use a streaming table to ingest raw data, and then create materialized views on top of the streaming table for downstream analytics.

## Gotchas

- **Not for all streaming**: Materialized views are not a direct replacement for all streaming workloads. For low-latency, continuous streaming, use streaming tables.
- **Refresh Costs**: While materialized views are updated incrementally, there is still a cost associated with each refresh. Be mindful of the refresh schedule.
- **Source Data**: Materialized views are designed to read from Delta Lake tables. While they can read from other sources, performance may not be optimal.

## Mock Questions

1.  **When would you choose to use a Streaming Table over a Materialized View?**
    a.  When you need to pre-compute the results of a complex query.
    b.  When you are ingesting data from an append-only streaming source and require low latency.
    c.  When you are working with batch data that changes infrequently.
    d.  When you need to perform complex aggregations.

2.  **What is a key characteristic of a Materialized View?**
    a.  It is designed for low-latency, continuous streaming.
    b.  It is always recomputed from scratch.
    c.  It is automatically and incrementally updated.
    d.  It can only be defined in Python.

3.  **Which of the following is a recommended best practice?**
    a.  Using Materialized Views for raw data ingestion.
    b.  Using Streaming Tables for complex aggregations.
    c.  Using a Streaming Table to ingest raw data and then creating Materialized Views on top of it.
    d.  Refreshing Materialized Views every second.

**Answers:**
1.  b
2.  c
3.  c

## References

[1] [Materialized Views and Streaming Tables](https://www.databricks.com/blog/introducing-materialized-views-and-streaming-tables-databricks-sql)
