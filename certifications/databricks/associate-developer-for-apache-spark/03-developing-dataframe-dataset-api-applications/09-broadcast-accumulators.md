# Broadcast Variables and Accumulators

## Overview

Describe broadcast variables (read-only shared data) and accumulators (write-only counters).

## Broadcast Variables

- Purpose: Efficiently distribute read-only lookup data to all executors.  
- Avoid repeated serialization per task.

```python
lookup = {"US": "United States", "CA": "Canada"}
broadcast_lookup = spark.sparkContext.broadcast(lookup)

def map_country(code):
    return broadcast_lookup.value.get(code, "Unknown")

from pyspark.sql.types import StringType
map_country_udf = F.udf(map_country, StringType())
df_mapped = df.withColumn("country_name", map_country_udf(F.col("country_code")))
```

## Accumulators

- Purpose: Aggregate metrics across tasks (e.g., error counts, row counts).  
- Write-only from tasks; read from driver after action.

```python
error_count = spark.sparkContext.accumulator(0)

def process_row(row):
    if row.status == "error":
        error_count.add(1)
    return row

df.foreach(lambda row: process_row(row))
print(f"Total errors: {error_count.value}")
```

## Best Practices

- Broadcast small lookup tables (<few MB).  
- Use accumulators for counters, not large data structures.  
- Access accumulator value only after action completes.

## Sample Questions

1. Purpose of broadcast variables?  
2. When use accumulators?  
3. Can tasks read accumulator values?  
4. Size limit for broadcast variables?  
5. Why broadcast instead of join for small lookups?

## Answers

1. Distribute read-only data efficiently to all executors.  
2. To track metrics (counts, sums) across distributed tasks.  
3. No; write-only from tasks, read from driver.  
4. Keep under few hundred MB; large sizes cause memory pressure.  
5. Avoids shuffle and reduces network overhead.

## References

- [Broadcast Variables](https://spark.apache.org/docs/latest/rdd-programming-guide.html#broadcast-variables)
- [Accumulators](https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators)

---

Previous: [UDFs](./08-user-defined-functions.md)  
Next: [Broadcast Joins](./10-broadcast-joins.md)
