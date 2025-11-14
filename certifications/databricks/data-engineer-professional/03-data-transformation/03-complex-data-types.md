# Complex Data Types

## Concept Definition

In addition to standard data types like strings, integers, and dates, Spark SQL supports complex data types that allow you to represent more intricate data structures. These complex data types are essential for working with semi-structured data, such as JSON and XML, and for creating more sophisticated data models. [1]

## Key Complex Data Types

- **`ARRAY`**: A sequence of elements of the same data type.
- **`MAP`**: A collection of key-value pairs. Keys must be of the same data type, and values must be of the same data type.
- **`STRUCT`**: A collection of named fields, similar to a struct in C or a row in a table.

## Python/PySpark Examples

### Working with `ARRAY`

```python
from pyspark.sql.functions import explode

# Explode an array into multiple rows
df.withColumn("exploded_array", explode(df.my_array)).show()
```

### Working with `MAP`

```python
from pyspark.sql.functions import map_keys

# Get the keys of a map
df.withColumn("map_keys", map_keys(df.my_map)).show()
```

### Working with `STRUCT`

```python
# Access a field within a struct
df.select("my_struct.my_field").show()
```

## Best Practices

- **Use `explode()` to Unnest Arrays**: The `explode()` function is the standard way to unnest an array into multiple rows.
- **Use Dot Notation for Structs**: Use dot notation to access fields within a struct.
- **Use Higher-Order Functions**: Use higher-order functions like `TRANSFORM` and `FILTER` to manipulate arrays and maps directly in SQL.

## Gotchas

- **`explode()` Creates Nulls**: If an array is null or empty, `explode()` will produce a null value. You may need to handle these nulls in your downstream logic.
- **Map Keys Must Be Unique**: The keys in a map must be unique.
- **Performance**: While powerful, complex data types can sometimes be less performant than simple data types. Consider the trade-offs between flexibility and performance.

## Mock Questions

1. **Which function would you use to convert an array of elements into multiple rows?**

    a. `transform()`  
    b. `explode()`  
    c. `filter()`  
    d. `select()`

2. **How do you access a field within a `STRUCT` type column?**

    a. Using bracket notation (e.g., `my_struct["my_field"]`)  
    b. Using dot notation (e.g., `my_struct.my_field`)  
    c. Using the `getField()` function  
    d. You cannot access individual fields within a struct.

3. **What is a key characteristic of a `MAP` data type?**

    a. It is an ordered sequence of elements.  
    b. It is a collection of key-value pairs.  
    c. It can only store string values.  
    d. It cannot be used in a DataFrame.

## Answers

1. b
2. b
3. b

## References

[1] [Data types | Databricks on AWS](https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-datatypes)
