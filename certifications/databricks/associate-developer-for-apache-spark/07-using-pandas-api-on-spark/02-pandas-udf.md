# Create and Invoke Pandas UDF

## Overview

Pandas UDFs vectorize operations using Apache Arrow for efficient batch processing.

## Types of Pandas UDFs

1. **Series to Series**: Scalar transformation.  
2. **Iterator of Series to Iterator of Series**: Memory-efficient batch processing.  
3. **Series to Scalar**: Grouped aggregation.  
4. **Multiple Series to Series**: Multi-column transformation.

## Example: Series to Series

```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

@pandas_udf("double")
def add_ten(values: pd.Series) -> pd.Series:
    return values + 10

df = spark.createDataFrame([(1,), (2,), (3,)], ["value"])
result = df.withColumn("value_plus_ten", add_ten("value"))
result.show()
```

## Example: Grouped Aggregation

```python
from pyspark.sql import functions as F

@pandas_udf("double")
def weighted_avg(values: pd.Series, weights: pd.Series) -> float:
    return (values * weights).sum() / weights.sum()

grouped = df.groupBy("region").agg(weighted_avg("amount", "weight").alias("weighted_avg"))
```

## Example: Iterator Pattern (Memory-Efficient)

```python
from typing import Iterator

@pandas_udf("double")
def scale_batch(iterator: Iterator[pd.Series]) -> Iterator[pd.Series]:
    for batch in iterator:
        yield batch * 2
```

## Enable Arrow

```python
spark.conf.set("spark.sql.execution.arrow.pyspark.enabled", "true")
```

## Best Practices

- Use Pandas UDFs instead of row-based UDFs for better performance.  
- Enable Arrow for 10-100x speedup on serialization.  
- Profile memory usage with large batches.

## Sample Questions

1. Difference between Pandas UDF and scalar UDF?  
2. How enable Arrow?  
3. Which UDF type for grouped aggregation?  
4. Performance benefit of Pandas UDF?  
5. When use iterator pattern?

## Answers

1. Pandas UDF processes batches (vectorized); scalar processes row-by-row.  
2. Set `spark.sql.execution.arrow.pyspark.enabled` to `true`.  
3. Series to Scalar UDF.  
4. 10-100x faster via Arrow serialization and vectorization.  
5. When processing large datasets to avoid loading entire partition in memory.

## References

- [Pandas UDF](https://spark.apache.org/docs/latest/api/python/user_guide/sql/arrow_pandas.html#pandas-udfs-aka-vectorized-udfs)

---

Previous: [Pandas API Advantages](./01-pandas-api-advantages.md)  
Next: [README](../README.md)
