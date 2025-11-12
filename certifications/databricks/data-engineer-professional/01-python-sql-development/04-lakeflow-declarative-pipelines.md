# Lakeflow Declarative Pipelines

## Concept Definition

Lakeflow Declarative Pipelines is a framework for creating batch and streaming data pipelines in SQL and Python. It simplifies ETL development by providing a declarative API, automatic orchestration, and built-in data quality features. With Lakeflow, you define the desired end state of your data, and the framework handles the complexities of data processing, including dependency management, error handling, and performance optimization. [1]

## Key Features

- **Declarative API**: Define your data pipelines using a simple, declarative syntax in SQL or Python.
- **Automatic Orchestration**: Lakeflow automatically manages the dependencies between your datasets and orchestrates the execution of your pipeline.
- **Incremental Processing**: Lakeflow intelligently processes only the new or changed data, reducing processing time and cost.
- **Data Quality**: Define data quality constraints and specify how to handle records that violate those constraints (e.g., drop, quarantine).
- **Error Handling**: Lakeflow provides robust error handling and retry mechanisms to ensure your pipelines are reliable.

## Python/PySpark Examples

### Defining a Lakeflow Pipeline in Python

```python
import dlt
from pyspark.sql.functions import *

@dlt.table(
  name="raw_data",
  comment="Raw data from JSON source",
  table_properties={"quality": "bronze"},
  partition_cols=["id"]
)
def raw_data():
  return (
    spark.readStream.format("cloudFiles")
      .option("cloudFiles.format", "json")
      .load("/path/to/raw/data")
  )

@dlt.table(
  comment="Cleaned and transformed data"
)
@dlt.expect_or_drop("valid_timestamp", "timestamp IS NOT NULL")
def cleaned_data():
  return (
    dlt.read_stream("raw_data")
      .withColumn("timestamp", to_timestamp("timestamp"))
      .select("id", "timestamp", "value")
  )
```

### Defining a Lakeflow Pipeline in SQL

```sql
-- Raw Data
CREATE STREAMING LIVE TABLE raw_data
COMMENT "Raw data from JSON source"
AS SELECT * FROM cloud_files("/path/to/raw/data", "json");

-- Cleaned Data
CREATE STREAMING LIVE TABLE cleaned_data
COMMENT "Cleaned and transformed data"
TBLPROPERTIES ("quality" = "expect_or_drop")
AS SELECT
  id,
  to_timestamp(timestamp) as timestamp,
  value
FROM STREAM(LIVE.raw_data)
WHERE timestamp IS NOT NULL;
```

## Best Practices

- **Use Autoloader**: For ingesting files from cloud storage, use Autoloader (`cloud_files`) to automatically discover and process new files.
- **Define Data Quality Rules**: Use expectations to define data quality rules and ensure the reliability of your data.
- **Separate Raw and Cleaned Data**: It's a good practice to separate your raw data from your cleaned and transformed data into different tables.
- **Use Comments**: Add comments to your tables to describe their purpose and schema.

## Gotchas

- **Streaming vs. Batch**: Understand the difference between `dlt.read()` for batch processing and `dlt.read_stream()` for streaming.
- **Table Naming**: Table names in Lakeflow are case-insensitive.
- **Circular Dependencies**: Be careful not to create circular dependencies between your tables, as this will cause your pipeline to fail.
- **Permissions**: Ensure that your pipeline has the necessary permissions to read from the source and write to the target.

## Mock Questions

1. **What is the primary benefit of the declarative nature of Lakeflow Declarative Pipelines?**

    a.  It allows you to write more complex code.  
    b.  It simplifies pipeline development by allowing you to define the end state of your data, while the framework handles the implementation details.  
    c.  It requires you to manually manage dependencies between your datasets.  
    d.  It only supports batch processing.  

2. **How does Lakeflow handle data quality?**

    a.  It automatically cleans all data.  
    b.  It ignores data quality issues.  
    c.  It allows you to define expectations and specify how to handle records that violate them.  
    d.  It sends an email to the data owner for every data quality issue.  

3. **What is the recommended way to ingest files from cloud storage in a Lakeflow pipeline?**

    a.  Use the `spark.read.format("json").load()` command.  
    b.  Use Autoloader (`cloud_files`).  
    c.  Write a custom Python script to list and read the files.  
    d.  Manually trigger the pipeline for each new file.  

**Answers:**
1. b
2. c
3. b

## References

[1] [Lakeflow Declarative Pipelines | Databricks on AWS](https://docs.databricks.com/aws/en/ldp/)
