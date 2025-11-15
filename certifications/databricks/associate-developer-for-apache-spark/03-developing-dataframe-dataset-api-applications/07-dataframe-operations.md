# DataFrame Operations: Sort, Iterate, Print Schema, Convert

## Overview

Perform common DataFrame introspection and transformation operations.

## Sorting

```python
# Ascending sort
sorted_asc = df.orderBy("amount")

# Descending sort
sorted_desc = df.orderBy(F.col("amount").desc())

# Multi-column sort
multi_sort = df.orderBy("region", F.col("amount").desc())
```

## Iteration

```python
# Iterate rows (avoid on large data; use collect sparingly)
for row in df.limit(10).collect():
    print(row.customer_id, row.amount)

# Map partitions (more efficient for partition-wide logic)
def process_partition(iterator):
    for row in iterator:
        yield (row.customer_id, row.amount * 1.1)

rdd_mapped = df.rdd.mapPartitions(process_partition)
```

## Print Schema

```python
df.printSchema()
```

## Conversion

```python
# DataFrame to list of Rows
rows = df.collect()

# DataFrame to Pandas (small data only)
pandas_df = df.limit(1000).toPandas()

# Pandas to Spark DataFrame
spark_df = spark.createDataFrame(pandas_df)

# DataFrame to RDD
rdd = df.rdd
```

## Best Practices

- Avoid `collect()` on large datasets; use `limit` or sampling.  
- Use `toPandas()` only for visualizations or small result sets.  
- Prefer DataFrame API over RDD for optimization benefits.

## Sample Questions

1. Sort DataFrame descending by column?  
2. Why avoid `collect()` on large data?  
3. How inspect DataFrame schema?  
4. Convert Spark DataFrame to Pandas?  
5. When use `rdd.mapPartitions`?

## Answers

1. `orderBy(F.col("column").desc())`.  
2. Pulls all data to driver memory causing OOM.  
3. `df.printSchema()`.  
4. `df.toPandas()`.  
5. For partition-level custom logic not expressible in DataFrame API.

## References

- [DataFrame API](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html)

---

Previous: [I/O Operations](./06-input-output-operations.md)  
Next: [User-Defined Functions (UDFs)](./08-user-defined-functions.md)
