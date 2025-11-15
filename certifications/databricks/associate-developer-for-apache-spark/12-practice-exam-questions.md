# Practice Exam Questions

## Overview

Representative multiple-choice and scenario-based questions covering core Spark exam domains. Answers with rationale included.

## Question Set

### 1. DataFrame Projection

Which approach best reduces scan IO on wide Parquet tables?  
a. Use `select` with only needed columns early.  
b. Apply `distinct` then filter.  
c. Cache full table then filter.  
d. Convert to RDD first.

### 2. Broadcast Join Selection

A dimension table has 50K rows (~4MB). The fact table has 500M rows. Optimal join pattern?  
a. Shuffle hash join.  
b. Broadcast dimension then join.  
c. Convert both to RDD and use `mapPartitions`.  
d. Use Cartesian join then filter.

### 3. Window Frame Semantics

What does `rowsBetween(-2, 0)` represent?  
a. Previous two rows plus current within partition ordering.  
b. Entire partition.  
c. Current row only.  
d. Future two rows.

### 4. Structured Streaming Watermark

Why apply a 15-minute watermark on an event-time aggregation?  
a. To guarantee no duplicates.  
b. To prune state for data older than 15 minutes late.  
c. To speed ingestion by skipping parse.  
d. To force complete mode output.

### 5. Catalyst Optimization Benefit

Using built-in functions instead of Python UDF yields:  
a. More shuffle partitions.  
b. No difference in execution.  
c. Better optimized physical plan and potential codegen.  
d. Streaming-only improvements.

### 6. Partition Pruning

Given partitioned path `/data/sales/date=YYYY-MM-DD/region=REGION/`, which filter maximizes pruning?  
a. `WHERE amount > 100`  
b. `WHERE date >= '2025-11-01' AND region = 'EMEA'`  
c. `WHERE substring(region,1,2) = 'EM'`  
d. `WHERE to_date(date) >= current_date()`

### 7. Repartition vs Coalesce

You need to reduce 200 partitions to 40 with minimal shuffle. What method?  
a. `repartition(40)`  
b. `coalesce(40)`  
c. `coalesce(400)`  
d. `repartition(400)`

### 8. Streaming Checkpoint Purpose

Checkpoint directory stores:  
a. Optimized Parquet schema only.  
b. State and progress metadata enabling recovery.  
c. Column statistics for broadcast.  
d. Window frames.

### 9. Detecting Join Skew

Primary indicator in Spark UI:  
a. Uniform task durations.  
b. A few tasks extremely longer than others reading huge shuffle blocks.  
c. Driver memory spike only.  
d. Fewer stages than expected.

### 10. Avoiding Driver OOM

Which is safer to inspect large data characteristics?  
a. `collect()` entire DataFrame.  
b. Sample & `toPandas()` small subset.  
c. Use broadcast hint.  
d. Convert to RDD and save locally.

### 11. Watermark Misconfiguration Result

Setting watermark larger than expected lateness:  
a. Causes system crash.  
b. Retains excessive state longer than needed.  
c. Eliminates all late rows.  
d. Forces complete mode always.

### 12. Multiple Aggregations Efficiently

Best practice:  
a. Separate `groupBy` calls per metric.  
b. Use one `groupBy().agg()` with all expressions.  
c. Convert to RDD reduceByKey for each.  
d. Cache before each aggregation.

### 13. Temp View Lifetime

Global temp view persists until:  
a. Spark application terminates.  
b. Next action call.  
c. First shuffle occurs.  
d. Explicit manual refresh.

### 14. Pandas UDF Use Case

Appropriate scenario:  
a. Simple string uppercase conversion.  
b. Heavy vector math on numeric columns.  
c. Basic equality filter.  
d. Column renaming.

### 15. Broadcast Threshold Control

Adjust broadcast threshold config to:  
a. Force all joins to shuffle.  
b. Permit larger small tables to be broadcast.  
c. Disable caching entirely.  
d. Change checkpoint retention.

## Answers

1. a  
2. b  
3. a  
4. b  
5. c  
6. b  
7. b  
8. b  
9. b  
10. b  
11. b  
12. b  
13. a  
14. b  
15. b  

## References

- [Spark SQL Programming Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [Structured Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [Performance Tuning Docs](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

---

Previous: [Error Handling, Debugging, and Common Pitfalls](./11-error-handling-debugging-and-common-pitfalls.md)
