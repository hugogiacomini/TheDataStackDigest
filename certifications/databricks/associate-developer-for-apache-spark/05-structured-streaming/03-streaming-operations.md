# Streaming Operations: Selection, Projection, Window, Aggregation

## Overview

Perform transformations on streaming DataFrames with stateful/stateless operations.

## Stateless Operations

```python
# Selection and projection
filtered = stream.select("user_id", "amount").filter(F.col("amount") > 100)

# Map transformations
enriched = stream.withColumn("amount_usd", F.col("amount") * 1.1)
```

## Windowed Aggregations

```python
from pyspark.sql import functions as F

# Tumbling window (5-minute fixed windows)
windowed = stream \
    .groupBy(F.window("timestamp", "5 minutes")) \
    .agg(F.sum("amount").alias("total_amount"))

# Sliding window (5-minute window, 1-minute slide)
sliding = stream \
    .groupBy(F.window("timestamp", "5 minutes", "1 minute")) \
    .agg(F.count("*").alias("event_count"))

# Session window (30-minute gap)
session = stream \
    .groupBy("user_id", F.session_window("timestamp", "30 minutes")) \
    .agg(F.sum("amount").alias("session_total"))
```

## Watermarking for Late Data

```python
# Drop events more than 10 minutes late
watermarked = stream \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(F.window("timestamp", "5 minutes")) \
    .agg(F.sum("amount"))
```

## Best Practices

- Use watermarks to limit state growth in aggregations.  
- Prefer tumbling windows for simplicity; sliding for overlapping analysis.  
- Monitor state size and checkpoint growth over time.

## Sample Questions

1. Difference between tumbling and sliding windows?  
2. Purpose of watermark?  
3. Which operations require state?  
4. How define 5-minute tumbling window?  
5. Why limit state growth?

## Answers

1. Tumbling: non-overlapping; Sliding: overlapping with slide interval.  
2. Defines how late data is tolerated before dropping.  
3. Aggregations, joins, deduplication.  
4. `F.window("timestamp", "5 minutes")`.  
5. Prevents memory exhaustion from unbounded state.

## References

- [Windowing](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#window-operations-on-event-time)

---

Previous: [Create & Write Streaming DataFrames](./02-create-write-streaming-dataframes.md)  
Next: [Streaming Deduplication](./04-streaming-deduplication.md)
