# Data Formats

## Concept Definition

Databricks supports a wide variety of data formats for reading and writing data. The choice of data format can have a significant impact on performance, storage costs, and the types of operations you can perform. Understanding the characteristics of different data formats is crucial for building efficient data pipelines. [1]

## Key Data Formats

| Format | Description | Use Case |
|---|---|---|
| **Delta Lake** | An open-source storage layer that brings ACID transactions to Apache Spark and big data workloads. It is the default format in Databricks. | The recommended format for all tables in Databricks. |
| **Parquet** | A columnar storage format that is highly optimized for analytical workloads. | A good choice for storing large datasets for analytical querying. |
| **ORC** | Another columnar storage format that is also optimized for analytical workloads. | Similar to Parquet, often used in Hadoop ecosystems. |
| **JSON** | A human-readable text format that is widely used for data exchange. | Ingesting data from APIs and other systems that produce JSON. |
| **CSV** | A simple text format for storing tabular data. | Ingesting data from legacy systems or spreadsheets. |
| **Avro** | A row-based storage format that is well-suited for data serialization. | Often used with Kafka and other messaging systems. |
| **Text** | A simple format for reading and writing plain text files. | Ingesting unstructured or semi-structured text data. |
| **Binary** | A format for reading and writing binary files. | Ingesting images, videos, and other binary data. |
| **XML** | A markup language for encoding documents in a format that is both human-readable and machine-readable. | Ingesting data from systems that produce XML. |

## Python/PySpark Examples

### Reading a Parquet File

```python
df = spark.read.format("parquet").load("/path/to/data.parquet")
```

### Writing a Delta Table

```python
df.write.format("delta").save("/path/to/delta/table")
```

### Reading a JSON File with Schema Inference

```python
df = spark.read.format("json").option("inferSchema", "true").load("/path/to/data.json")
```

## Best Practices

- **Use Delta Lake as the Default**: For all tables in Databricks, use the Delta Lake format to take advantage of its features like ACID transactions, time travel, and schema evolution.
- **Use Columnar Formats for Analytics**: For large-scale analytical workloads, use columnar formats like Parquet or ORC for better performance.
- **Specify the Schema**: When reading data from formats like JSON and CSV, it is a best practice to specify the schema to avoid the performance overhead of schema inference.

## Gotchas

- **Schema Inference is Slow**: Schema inference can be slow and may not always infer the correct data types. It is best to explicitly define the schema when possible.
- **CSV Complexity**: CSV files can be complex to parse due to issues like delimiters, quotes, and newlines. Be sure to configure the CSV reader options correctly.
- **Nested JSON**: When working with nested JSON data, you may need to use functions like `from_json` and `explode` to flatten the data into a tabular format.

## Mock Questions

1. **What is the default and recommended data format for tables in Databricks?**

    a.  Parquet  
    b.  CSV  
    c.  Delta Lake  
    d.  JSON  

2. **Why is it a best practice to specify the schema when reading data from formats like JSON and CSV?**

    a.  To make the code more complex.  
    b.  To avoid the performance overhead of schema inference and ensure data type correctness.  
    c.  To slow down the data ingestion process.  
    d.  Because it is required by the Spark API.  

3. **Which data format is best suited for storing large datasets for analytical querying?**

    a.  JSON
    b.  CSV
    c.  Parquet
    d.  Text

## Answers

1. c
2. b
3. c

## References

[1] [Data format options | Databricks on AWS](https://docs.databricks.com/aws/en/query/formats/)
