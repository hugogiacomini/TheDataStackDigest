# Data Deduplication and Validation

## Overview

Remove duplicates and validate data integrity with DataFrame operations.

## Deduplication

```python
# Drop exact duplicate rows
unique = df.dropDuplicates()

# Drop duplicates on specific columns
unique_key = df.dropDuplicates(["customer_id", "order_date"])

# Keep first occurrence within window (advanced)
from pyspark.sql.window import Window
w = Window.partitionBy("customer_id").orderBy("timestamp")
deduplicated = df.withColumn("rn", F.row_number().over(w)).filter("rn = 1").drop("rn")
```

## Validation

```python
# Null checks
non_null = df.filter(F.col("email").isNotNull())

# Range validation
valid_amounts = df.filter((F.col("amount") >= 0) & (F.col("amount") < 10000))

# Regex validation
valid_emails = df.filter(F.col("email").rlike("^[^@]+@[^@]+\\.[^@]+$"))

# Count violations
violation_count = df.filter(~F.col("status").isin(["active", "pending"])).count()
```

## Best Practices

- Deduplicate early to reduce downstream processing.  
- Use window functions for tie-breaking (e.g., most recent record).  
- Log or quarantine invalid records for investigation.

## Sample Questions

1. How remove duplicates based on key columns?  
2. When use window row_number for dedup?  
3. Validate email format in DataFrame?  
4. Check for null values?  
5. Why quarantine invalid records?

## Answers

1. `dropDuplicates(["col1", "col2"])`.  
2. When need to keep specific occurrence (first/last by order).  
3. Use `rlike` with regex pattern.  
4. `filter(col.isNotNull())` or `filter(col.isNull())`.  
5. Preserve audit trail and enable root cause analysis.

## References

- [DataFrame Deduplication](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.dropDuplicates.html)

---

Previous: [Column & Row Manipulation](./01-column-row-manipulation.md)  
Next: [Aggregate Operations](./03-aggregate-operations.md)
