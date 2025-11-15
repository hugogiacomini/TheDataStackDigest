# Structured Streaming Basics

## Overview

Introduce micro-batch execution model, sources/sinks, watermarking, and common operations for incremental computation.

## Prerequisites

- Batch DataFrame operations; JSON/Parquet reading.

## Concepts

- Trigger: Micro-batch scheduling; default continuous micro-batches.
- Checkpointing: State & progress persistence; required for exactly-once semantics.
- Watermark: Late data handling; trims state beyond threshold.
- Output modes: append, complete, update.

## Hands-on Walkthrough

### File Stream Ingestion (PySpark)

```python
from pyspark.sql.functions import window, col

stream_df = (spark.readStream.format("json")
             .schema("user_id STRING, amount DOUBLE, event_ts TIMESTAMP")
             .load("/mnt/stream/incoming/") )

agg = (stream_df
       .withWatermark("event_ts", "10 minutes")
       .groupBy(window(col("event_ts"), "5 minutes"), col("user_id"))
       .sum("amount") )

query = (agg.writeStream
             .format("parquet")
             .option("checkpointLocation", "/mnt/ckpt/amounts/")
             .outputMode("append")
             .start("/mnt/stream/output/") )
```

### Console Sink for Debugging

```python
( stream_df.writeStream
  .format("console")
  .option("truncate", False)
  .start() )
```

## Production Considerations

- Unique checkpoints per query; never share directories.
- Watermark tuning: Balance late data tolerance vs state size.
- Backpressure: Monitor query progress; adjust ingestion rate or cluster size.

## Troubleshooting

- Missing output: Check watermark logic vs event times; verify checkpoint path.
- State growth: Narrow group keys; apply watermarks; clean old state.
- Reprocessing duplication: Avoid deleting checkpoints; handle restarts gracefully.

## Sample Questions

1. What does a watermark accomplish?  
2. Why checkpointing required?  
3. Difference between append and complete modes?  
4. How to debug early during development?  
5. Why isolate checkpoint directories?

## Answers

1. Defines threshold to drop late data & prune aggregation state.  
2. Enables progress/state recovery ensuring exactly-once semantics.  
3. Append outputs only new rows; complete outputs entire aggregated result each batch.  
4. Use console sink for immediate feedback.  
5. Prevents state corruption across unrelated queries.

## References

- [Structured Streaming Programming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [Watermarking](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#handling-late-data-and-watermarking)

---

Previous: [Spark SQL and Temp Views](./09-spark-sql-and-temp-views.md)  
Next: [Error Handling, Debugging, and Common Pitfalls](./11-error-handling-debugging-and-common-pitfalls.md)
