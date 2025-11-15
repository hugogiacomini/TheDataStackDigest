# Create and Write Streaming DataFrames/Datasets

## Overview

Read from streaming sources and write to sinks with proper configurations.

## Streaming Sources

```python
# File source (JSON, Parquet, CSV, Delta)
stream = spark.readStream.format("json").schema(schema).load("/mnt/input/")

# Kafka source
kafka_stream = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "broker:9092") \
    .option("subscribe", "topic1") \
    .load()

# Rate source (testing)
rate_stream = spark.readStream.format("rate").option("rowsPerSecond", 10).load()
```

## Streaming Sinks

```python
# Console sink (debugging)
query = stream.writeStream.format("console").outputMode("append").start()

# File sink (Parquet)
query = stream.writeStream \
    .format("parquet") \
    .option("checkpointLocation", "/mnt/checkpoints/") \
    .option("path", "/mnt/output/") \
    .start()

# Delta sink (production)
query = stream.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/mnt/checkpoints/") \
    .outputMode("append") \
    .start("/mnt/delta/table/")

# Kafka sink
query = stream.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)") \
    .writeStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "broker:9092") \
    .option("topic", "output_topic") \
    .option("checkpointLocation", "/mnt/checkpoints/") \
    .start()
```

## Best Practices

- Always set checkpoint location for fault tolerance.  
- Use Delta for ACID and schema evolution.  
- Test with console sink before production deployment.

## Sample Questions

1. How create streaming DataFrame from files?  
2. Which sinks support exactly-once?  
3. Purpose of checkpoint in streaming?  
4. Kafka source required options?  
5. When use rate source?

## Answers

1. `spark.readStream.format("format").schema(schema).load("path")`.  
2. Delta, Kafka (with idempotent producer), File sinks.  
3. Enables state recovery and exactly-once semantics.  
4. `kafka.bootstrap.servers`, `subscribe` (or `assign`).  
5. For testing/benchmarking streaming logic.

## References

- [Streaming Sources & Sinks](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#input-sources)

---

Previous: [Streaming Engine](./01-streaming-engine-explanation.md)  
Next: [Streaming Operations](./03-streaming-operations.md)
