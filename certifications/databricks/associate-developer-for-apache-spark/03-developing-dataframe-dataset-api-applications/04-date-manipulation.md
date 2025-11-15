# Date Data Type Manipulation

## Overview

Convert Unix epoch to date strings, extract date components, perform date arithmetic.

## Common Date Operations

```python
from pyspark.sql import functions as F

# Unix timestamp to date
df_with_date = df.withColumn("event_date", F.from_unixtime(F.col("epoch_ts")))

# String to date
df_parsed = df.withColumn("order_date", F.to_date(F.col("date_string"), "yyyy-MM-dd"))

# Extract date components
df_components = df.withColumn("year", F.year("event_date")) \
                  .withColumn("month", F.month("event_date")) \
                  .withColumn("day", F.dayofmonth("event_date")) \
                  .withColumn("dow", F.dayofweek("event_date"))

# Date arithmetic
df_shifted = df.withColumn("next_week", F.date_add(F.col("event_date"), 7))
df_diff = df.withColumn("days_since", F.datediff(F.current_date(), F.col("event_date")))

# Current timestamp
df_stamped = df.withColumn("load_ts", F.current_timestamp())
```

## Best Practices

- Store dates as `DateType` or `TimestampType` for efficient filtering.  
- Use ISO format strings for clarity.  
- Partition tables by date columns for time-range pruning.

## Sample Questions

1. Convert Unix epoch to date?  
2. Extract year from a date column?  
3. Compute days between two dates?  
4. How add days to a date?  
5. Why store as date type vs string?

## Answers

1. `from_unixtime(col)` or `to_timestamp(col)`.  
2. `F.year(col)`.  
3. `F.datediff(end_date, start_date)`.  
4. `F.date_add(col, n)`.  
5. Enables date comparisons, arithmetic, and partition pruning.

## References

- [Date Functions](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html#datetime-functions)

---

Previous: [Aggregate Operations](./03-aggregate-operations.md)  
Next: [Combine DataFrames: Joins](./05-combine-dataframes-joins.md)
