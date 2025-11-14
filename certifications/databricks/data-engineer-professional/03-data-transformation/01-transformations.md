# Data Transformations

## Concept Definition

Data transformation is the process of converting data from one format or structure to another. In Databricks, this typically involves using Apache Spark to apply a series of transformations to a DataFrame to clean, enrich, and structure the data for a specific purpose. Transformations in Spark are lazy, meaning they are not executed until an action is called. [1]

## Key Transformation Functions

- **`withColumn()`**: Adds a new column or replaces an existing column.
- **`select()`**: Selects a set of columns.
- **`filter()`**: Filters rows based on a condition.
- **`groupBy()`**: Groups the DataFrame by one or more columns.
- **`agg()`**: Performs aggregations on a grouped DataFrame.
- **`join()`**: Joins two DataFrames together.
- **`union()`**: Combines two DataFrames with the same schema.

## Python/PySpark Examples

### Adding a New Column

```python
from pyspark.sql.functions import lit

df = df.withColumn("new_column", lit("new_value"))
```

### Filtering Data

```python
from pyspark.sql.functions import col

df = df.filter(col("my_column") > 100)
```

### Performing an Aggregation

```python
from pyspark.sql.functions import sum

df_agg = df.groupBy("category").agg(sum("value").alias("total_value"))
```

## Best Practices

- **Chain Transformations**: Chain transformations together to create a clear and concise data processing pipeline.
- **Use Built-in Functions**: Use Spark's built-in functions whenever possible, as they are highly optimized.
- **Cache Intermediate DataFrames**: If you are reusing a DataFrame multiple times, consider caching it in memory to improve performance.
- **Use the `transform` Function**: The `DataFrame.transform()` function provides a concise way to chain custom transformations. [2]

## Gotchas

- **Lazy Evaluation**: Remember that transformations are lazy. You need to call an action (e.g., `show()`, `count()`, `write()`) to trigger the execution of the transformations.
- **Data Skew**: Data skew can cause performance issues during transformations, especially with joins and aggregations. You may need to use techniques like salting to mitigate skew.
- **Null Values**: Be mindful of how null values are handled in your transformations. Use functions like `isNull()`, `isNotNull()`, and `na.fill()` to handle nulls explicitly.

## Mock Questions

1. **What does it mean that transformations in Spark are "lazy"?**

    a. They are slow to execute.  
    b. They are not executed until an action is called.  
    c. They are easy to write.  
    d. They can only be used with small datasets.

2. **Which function would you use to add a new column to a DataFrame?**

    a. `select()`  
    b. `filter()`  
    c. `withColumn()`  
    d. `groupBy()`

3. **What is a common technique to mitigate data skew during a join operation?**

    a. Using a smaller cluster.  
    b. Salting the join key.  
    c. Using the `union()` function.  
    d. Ignoring the skewed data.

## Answers

1. b
2. c
3. b

## References

[1] [What is data transformation on Azure Databricks?](https://docs.azure.cn/en-us/databricks/data-engineering/data-transformation)
[2] [pyspark.sql.DataFrame.transform](https://api-docs.databricks.com/python/pyspark/latest/pyspark.sql/api/pyspark.sql.DataFrame.transform.html)
