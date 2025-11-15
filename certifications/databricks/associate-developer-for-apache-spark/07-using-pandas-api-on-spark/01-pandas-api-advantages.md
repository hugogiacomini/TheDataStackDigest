# Pandas API on Spark: Advantages

## Overview

Pandas API on Spark (formerly Koalas) enables Pandas-like syntax on distributed Spark DataFrames.

## Key Advantages

1. **Familiar Syntax**: Write Pandas code that scales to big data.  
2. **No Rewrite**: Migrate existing Pandas scripts with minimal changes.  
3. **Spark Backend**: Leverage Catalyst optimizer and distributed execution.  
4. **Interoperability**: Convert between Pandas and Spark seamlessly.

## Example

```python
import pyspark.pandas as ps

# Read as Pandas-on-Spark DataFrame
psdf = ps.read_csv("s3://bucket/large_file.csv")

# Pandas-like operations
filtered = psdf[psdf['amount'] > 100]
grouped = psdf.groupby('region')['amount'].sum()

# Convert to Spark DataFrame for advanced operations
spark_df = psdf.to_spark()

# Convert back to Pandas (for small results)
pandas_df = psdf.to_pandas()
```

## Limitations

- Not all Pandas operations supported (check compatibility matrix).  
- Some operations trigger expensive computation (e.g., sorting entire dataset).  
- Performance may differ from native Spark DataFrame API.

## Best Practices

- Use for quick migration of Pandas workflows to big data.  
- Profile and optimize critical paths with native Spark API.  
- Limit `to_pandas()` calls to small result sets to avoid OOM.

## Sample Questions

1. What is Pandas API on Spark?  
2. Main benefit over pure Pandas?  
3. How convert to Spark DataFrame?  
4. Limitation of Pandas-on-Spark?  
5. When use Pandas API on Spark?

## Answers

1. Pandas-compatible API backed by distributed Spark.  
2. Scales to large datasets beyond single-machine memory.  
3. `.to_spark()`.  
4. Not all Pandas operations supported; some may be slow.  
5. Migrating existing Pandas code to big data; rapid prototyping.

## References

- [Pandas API on Spark](https://spark.apache.org/docs/latest/api/python/user_guide/pandas_on_spark/index.html)

---

Previous: [Deployment Modes](../06-spark-connect/02-deployment-modes.md)  
Next: [Pandas UDF](./02-pandas-udf.md)
