
# Append-Only Pipelines

## Concept Definition

An append-only pipeline is a data pipeline that only adds new records to a target table without modifying or deleting existing records. This is a common pattern for ingesting data from sources that are themselves append-only, such as log files, streaming data, and transactional systems. In Databricks, append-only pipelines are typically implemented using Structured Streaming or Lakeflow Declarative Pipelines. [1]

## Key Features

- **Simplicity**: Append-only pipelines are generally simpler to implement and manage than pipelines that require updates and deletes.
- **Performance**: Appending data is a highly optimized operation in Delta Lake, making append-only pipelines very performant.
- **Auditability**: Because data is never modified or deleted, append-only tables provide a complete audit trail of all data that has been ingested.

## Python/PySpark Examples

### Append-Only Pipeline with Structured Streaming

```python
# Read from a streaming source
stream_df = (
    spark.readStream.format("kafka")
    .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
    .option("subscribe", "my_topic")
    .load()
)

# Write to a Delta table in append mode
query = (
    stream_df.writeStream.format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/path/to/checkpoint")
    .start("/path/to/target/table")
)
```

### Append-Only Pipeline with Lakeflow Declarative Pipelines

In Lakeflow, the default flow for a streaming table is an append flow.

```python
import dlt

@dlt.table
def my_append_only_pipeline():
  return (
    spark.readStream.format("cloudFiles")
      .option("cloudFiles.format", "json")
      .load("/path/to/source/data")
  )
```

## Best Practices

- **Use `outputMode("append")`**: When using Structured Streaming, explicitly set the output mode to "append" to ensure that only new records are added to the target table.
- **Use Streaming Tables in Lakeflow**: In Lakeflow Declarative Pipelines, use streaming tables for append-only workloads.
- **Partition Your Data**: For large append-only tables, partition the data by a date or timestamp column to improve query performance.

## Gotchas

- **Duplicates**: If your source data may contain duplicates, you will need to implement a deduplication step in your pipeline.
- **Late-Arriving Data**: Append-only pipelines can be sensitive to late-arriving data. You may need to use watermarking to handle late data correctly.
- **Schema Changes**: If the schema of your source data changes, you will need to update your pipeline to handle the new schema.

## Mock Questions

1. **What is the default output mode for a Structured Streaming query?**

    a.  Append  
    b.  Complete  
    c.  Update  
    d.  There is no default output mode.  

2. **In Lakeflow Declarative Pipelines, what is the default flow for a streaming table?**

    a.  An append flow  
    b.  A merge flow  
    c.  A delete flow  
    d.  There is no default flow.  

3. **What is a potential issue with append-only pipelines?**

    a.  They are too slow.  
    b.  They are too complex.  
    c.  They may not handle duplicate records correctly without a deduplication step.  
    d.  They cannot be used with streaming data.  

## Answers

1. a
2. a
3. c

## References

[1] [Data is processed on Lakeflow Declarative Pipelines with flows](https://docs.databricks.com/aws/en/ldp/flows)

---

Next: [Schema Evolution and Expectations](./04-schema-evolution-and-expectations.md)
