# User-Defined Functions (UDFs)

## Concept Definition

A User-Defined Function (UDF) is a feature of Spark SQL that allows you to define your own functions in Python or Scala and use them in your data processing code. UDFs are used to extend the built-in functionality of Spark and perform custom transformations on your data. [1]

There are two main types of Python UDFs in Databricks:

- **Python UDFs (Scalar UDFs)**: These UDFs operate on a single row of data at a time. They are easy to implement but can have performance overhead due to serialization and deserialization of data between the Python and JVM processes.

- **Pandas UDFs (Vectorized UDFs)**: These UDFs use Apache Arrow to transfer data and pandas to work with the data. They operate on a series of data at a time, which can significantly improve performance compared to scalar UDFs. [2]

## Python/PySpark Examples

### Python UDF (Scalar)

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

# Define the Python function
def uppercase_udf(s):
  if s is not None:
    return s.upper()

# Register the UDF
uppercase = udf(uppercase_udf, StringType())

# Create a DataFrame
data = [("hello",), ("world",)]
df = spark.createDataFrame(data, ["text"])

# Use the UDF
df.withColumn("upper_text", uppercase(df.text)).show()
```

### Pandas UDF (Vectorized)

```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

# Define the Pandas UDF
@pandas_udf("string")
def uppercase_pandas_udf(s: pd.Series) -> pd.Series:
    return s.str.upper()

# Create a DataFrame
data = [("hello",), ("world",)]
df = spark.createDataFrame(data, ["text"])

# Use the Pandas UDF
df.withColumn("upper_text", uppercase_pandas_udf(df.text)).show()
```

## Best Practices

- **Use Built-in Functions When Possible**: Before writing a UDF, check if the desired functionality can be achieved using Spark's built-in functions. Built-in functions are almost always more performant than UDFs.
- **Prefer Pandas UDFs**: When you need to write a UDF, prefer Pandas UDFs over scalar Python UDFs for better performance.
- **Use `spark.sql.function.register()`**: To make your UDFs available in SQL queries, register them using `spark.sql.function.register()`.
- **Handle Nulls**: Always consider how your UDF will handle null values. The examples above include a null check.
- **Deterministic UDFs**: If your UDF is deterministic (i.e., it always returns the same output for the same input), mark it as such for potential optimization by Spark.

## Gotchas

- **Performance Overhead**: Scalar Python UDFs can be slow due to the cost of serializing and deserializing data between the JVM and Python processes for each row.
- **Data Skew**: If your data is skewed, some partitions may take much longer to process than others, especially with UDFs.
- **Complex Logic**: Avoid putting overly complex logic in UDFs. It can make your code harder to debug and maintain.
- **Type Hinting**: For Pandas UDFs, it's important to use correct type hints for the input and output series.

## Mock Questions

1. **What is the main advantage of using Pandas UDFs over scalar Python UDFs?**

    a.  They are easier to write.  
    b.  They can be used in SQL queries.  
    c.  They offer significantly better performance due to vectorized execution.  
    d.  They can handle null values automatically.  

2. **When should you consider writing a UDF?**

    a.  For any data transformation.  
    b.  When the desired functionality is not available in Spark's built-in functions.  
    c.  To make your code more complex.  
    d.  When you want to slow down your data processing.  

3. **What is a potential issue with using scalar Python UDFs?**

    a.  They are too fast.  
    b.  They can only be used with Pandas DataFrames.  
    c.  They can introduce performance bottlenecks due to serialization/deserialization overhead.  
    d.  They cannot be registered for use in SQL queries.  

## Answers

1. c
2. b
3. c

## References

[1] [User-defined scalar functions - Python | Databricks on AWS](https://docs.databricks.com/aws/en/udf/python)
[2] [pandas user-defined functions | Databricks on AWS](https://docs.databricks.com/aws/en/udf/pandas)
