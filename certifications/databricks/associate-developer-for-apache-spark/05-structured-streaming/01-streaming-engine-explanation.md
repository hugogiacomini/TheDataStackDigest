# Structured Streaming Engine Explanation

## Overview

Structured Streaming provides scalable, fault-tolerant stream processing using DataFrame API.

## Core Concepts

- **Micro-batches**: Processes data in small batches (default trigger interval).  
- **Continuous Processing**: Experimental low-latency mode (millisecond latency).  
- **Checkpointing**: Ensures exactly-once semantics via state recovery.

## Architecture

```text
Input Source → Streaming DataFrame → Transformations → Output Sink
                                        ↓
                               Checkpoint Location
```

## Example

```python
# Read stream from Kafka
stream_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "events") \
    .load()

# Parse and transform
from pyspark.sql import functions as F
parsed = stream_df.select(F.from_json(F.col("value").cast("string"), schema).alias("data")).select("data.*")

# Write stream to Delta
query = parsed.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/mnt/checkpoints/events/") \
    .outputMode("append") \
    .start("/mnt/delta/events/")

query.awaitTermination()
```

## Output Modes

- **Append**: Only new rows added to result (default).  
- **Complete**: Entire result table rewritten (for aggregations).  
- **Update**: Only updated rows written (requires state).

## Best Practices

- Always specify checkpoint location for fault tolerance.  
- Use Delta Lake as sink for ACID guarantees.  
- Monitor streaming query progress via Spark UI and metrics.

## Sample Questions

1. What is Structured Streaming?  
2. Difference between micro-batch and continuous?  
3. Purpose of checkpoint location?  
4. Three output modes?  
5. Why use Delta as streaming sink?

## Answers

1. Fault-tolerant stream processing using DataFrame API.  
2. Micro-batch trades latency for throughput; continuous offers low latency.  
3. Enables exactly-once processing and state recovery.  
4. Append, Complete, Update.  
5. ACID guarantees, schema evolution, time travel.

## References

- [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)

---

Previous: [Logging & Monitoring](../04-troubleshooting-and-tuning/03-logging-monitoring.md)  
Next: [Create and Write Streaming DataFrames](./02-create-write-streaming-dataframes.md)
