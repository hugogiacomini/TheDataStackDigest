# Streaming Deduplication with/without Watermark

## Overview

Remove duplicate records in streaming DataFrames using stateful deduplication.

## Basic Deduplication (Without Watermark)

```python
# Deduplicate on key columns (state grows unbounded)
deduplicated = stream.dropDuplicates(["event_id", "user_id"])
```

**Warning**: State size grows indefinitely; not production-safe for long-running streams.

## Deduplication with Watermark

```python
# Drop duplicates with bounded state using watermark
deduplicated = stream \
    .withWatermark("timestamp", "1 hour") \
    .dropDuplicates(["event_id", "user_id"])

# Write to sink
query = deduplicated.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/mnt/checkpoints/dedup/") \
    .outputMode("append") \
    .start("/mnt/delta/dedup/")
```

**Behavior**: Events older than watermark are dropped; state is purged for old keys.

## Advanced: Window-Based Deduplication

```python
# Deduplicate within hourly windows
from pyspark.sql import functions as F

windowed_dedup = stream \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(F.window("timestamp", "1 hour"), "event_id") \
    .agg(F.first("user_id").alias("user_id"), F.first("amount").alias("amount"))
```

## Best Practices

- Always use watermark for long-running production streams.  
- Set watermark delay based on expected late arrival time.  
- Monitor checkpoint size to validate state cleanup.

## Sample Questions

1. Risk of deduplication without watermark?  
2. How enable bounded deduplication?  
3. What happens to late events beyond watermark?  
4. Typical watermark delay value?  
5. Where is deduplication state stored?

## Answers

1. Unbounded state growth leading to OOM.  
2. Add `withWatermark` before `dropDuplicates`.  
3. They are dropped (not processed).  
4. Depends on SLA; commonly 10 minutes to 1 hour.  
5. Checkpoint location managed by Spark.

## References

- [Deduplication](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#deduplication)

---

Previous: [Streaming Operations](./03-streaming-operations.md)  
Next: [Spark Connect Features](../06-spark-connect/01-spark-connect-features.md)
