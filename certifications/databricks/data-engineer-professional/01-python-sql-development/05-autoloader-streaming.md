# Autoloader and Streaming

## Concept Definition

Autoloader is a feature in Databricks that allows you to incrementally and efficiently process new data files as they arrive in cloud storage. It provides a Structured Streaming source called `cloudFiles` that automatically processes new files as they arrive, with the option of also processing existing files in that directory. [1]

## Key Features

- **Scalability**: Autoloader can discover billions of files efficiently and supports near real-time ingestion of millions of files per hour.
- **Performance**: The cost of discovering files with Autoloader scales with the number of files being ingested, not the number of directories.
- **Schema Inference and Evolution**: Autoloader can automatically detect the schema of your data and handle schema changes over time.
- **Cost-Effective**: Autoloader uses native cloud APIs and file notification services to reduce the cost of file discovery.
- **Exactly-Once Guarantees**: Autoloader uses a checkpoint location to track ingestion progress and provide exactly-once data processing guarantees.

## Python/PySpark Examples

### Using Autoloader with Structured Streaming

```python
# Set up the stream
stream_df = (
    spark.readStream.format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/path/to/schema/location")
    .load("/path/to/source/data")
)

# Write the stream to a Delta table
query = (
    stream_df.writeStream.format("delta")
    .option("checkpointLocation", "/path/to/checkpoint/location")
    .start("/path/to/target/table")
)
```

### Using Autoloader in Lakeflow Declarative Pipelines

```python
import dlt

@dlt.table
def raw_data():
  return (
    spark.readStream.format("cloudFiles")
      .option("cloudFiles.format", "json")
      .load("/path/to/source/data")
  )
```

## Best Practices

- **Use with Lakeflow Declarative Pipelines**: Databricks recommends using Autoloader within Lakeflow Declarative Pipelines for incremental data ingestion. Lakeflow automatically manages schema and checkpoint locations.
- **Schema Location**: Always specify a schema location (`cloudFiles.schemaLocation`) to enable schema inference and evolution.
- **File Notification Mode**: For large numbers of files, use file notification mode to reduce file discovery costs. This requires additional cloud permissions.
- **Checkpoint Location**: Ensure that your checkpoint location is a persistent and reliable storage location.

## Gotchas

- **Permissions**: Autoloader requires appropriate permissions to read from the source directory and, if using file notification mode, to manage notification services.
- **Backfills**: When processing a large number of existing files (a backfill), it can take time for Autoloader to discover all the files. You can perform backfills asynchronously to avoid wasting compute resources.
- **Schema Mismatches**: If the schema of your data changes in a way that is not compatible with the existing schema, your stream may fail. Autoloader provides options for handling schema evolution.

## Mock Questions

1.  **What is the primary function of the `cloudFiles.schemaLocation` option in Autoloader?**
    a.  To specify the location of the source data.
    b.  To specify the location of the target table.
    c.  To enable schema inference and evolution by storing the schema information.
    d.  To specify the location of the checkpoint file.

2.  **Which of the following is a key benefit of Autoloader over the traditional file source in Structured Streaming?**
    a.  It only supports batch processing.
    b.  It is less scalable.
    c.  It can automatically detect and handle schema changes.
    d.  It requires you to manually manage file discovery.

3.  **What are the two file detection modes supported by Autoloader?**
    a.  Directory listing and file notification.
    b.  Manual and automatic.
    c.  Batch and streaming.
    d.  Full and incremental.

**Answers:**
1.  c
2.  c
3.  a

## References

[1] [What is Auto Loader? | Databricks on AWS](https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/)
