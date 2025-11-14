# Ingestion Sources

## Concept Definition

Databricks provides a variety of ways to ingest data from a wide range of sources. The best method for data ingestion depends on the data source, the data format, and the desired latency of the data pipeline. [1]

## Key Ingestion Sources

| Source | Description | Ingestion Method |
|---|---|---|
| **Cloud Object Storage** | Files in cloud storage like Amazon S3, Azure Data Lake Storage (ADLS) Gen2, and Google Cloud Storage (GCS). | Autoloader, `spark.read` |
| **Databases** | Relational databases like PostgreSQL, MySQL, and SQL Server. | JDBC connector |
| **Message Queues** | Streaming data from message queues like Apache Kafka and Azure Event Hubs. | Structured Streaming connectors |
| **SaaS Applications** | Data from SaaS applications like Salesforce and Workday. | Lakeflow Connect managed connectors |
| **Local Files** | Files on your local machine. | Databricks UI, `dbutils.fs.cp` |

## Python/PySpark Examples

### Ingesting from Cloud Object Storage with Autoloader

```python
df = (
    spark.readStream.format("cloudFiles")
    .option("cloudFiles.format", "json")
    .load("s3://my-bucket/my-data")
)
```

### Ingesting from a PostgreSQL Database

```python
df = (
    spark.read.format("jdbc")
    .option("url", "jdbc:postgresql://host:port/database")
    .option("dbtable", "my_table")
    .option("user", "my_user")
    .option("password", "my_password")
    .load()
)
```

### Ingesting from Kafka

```python
df = (
    spark.readStream.format("kafka")
    .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
    .option("subscribe", "my_topic")
    .load()
)
```

## Best Practices

- **Use Autoloader for Cloud Storage**: For ingesting files from cloud object storage, Autoloader is the recommended approach as it is scalable, efficient, and handles schema evolution.
- **Use Lakeflow Connect for SaaS Applications**: For ingesting data from SaaS applications, use the managed connectors in Lakeflow Connect for a simplified and automated experience.
- **Securely Store Credentials**: When connecting to databases and other systems, use Databricks secrets to securely store your credentials.

## Gotchas

- **JDBC Driver**: You may need to install the appropriate JDBC driver for your database on your cluster.
- **Network Connectivity**: Ensure that your Databricks workspace has network connectivity to the data source.
- **Data Volume**: For large data volumes, consider using a parallel ingestion method to improve performance.

## Mock Questions

1. **What is the recommended method for ingesting files from cloud object storage in Databricks?**

    a.  Using the JDBC connector.  
    b.  Using Autoloader.  
    c.  Writing a custom Python script.  
    d.  Manually uploading the files.  

2. **How should you securely store credentials for connecting to a database?**

    a.  Hardcode them in your notebook.  
    b.  Store them in a plain text file.  
    c.  Use Databricks secrets.  
    d.  Email them to your colleagues.  

3. **Which Databricks feature simplifies the ingestion of data from SaaS applications like Salesforce?**

    a.  Autoloader  
    b.  The JDBC connector  
    c.  Lakeflow Connect  
    d.  Structured Streaming  

## Answers

1. b
2. c
3. c

## References

[1] [Connect to data sources and external services | Databricks on AWS](https://docs.databricks.com/aws/en/connect/)
