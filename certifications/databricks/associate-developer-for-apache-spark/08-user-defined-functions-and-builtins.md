# User-Defined Functions and Built-ins

## Overview

Contrast built-in functions with UDFs/UDAFs; discuss performance implications and serialization concerns.

## Prerequisites

- Basic Python/Scala function creation.

## Concepts

- Built-in functions: Catalyst-aware, optimized codegen paths.
- Python UDF: Executes in Python worker; serialization overhead.
- Pandas UDF: Vectorized; leverages Arrow for performance.
- UDAF: Custom aggregation logic; more complex state management.

## Hands-on Walkthrough

### Built-in vs Regular UDF (PySpark)

```python
from pyspark.sql import functions as F
from pyspark.sql.types import StringType

# Built-in upper
built = spark.createDataFrame([(1, "alpha")], ["id", "txt"]).select(F.upper("txt"))

# Scalar UDF
@F.udf(returnType=StringType())
def to_upper(x: str) -> str:
    return x.upper() if x else x

udf_df = spark.createDataFrame([(1, "alpha")], ["id", "txt"]).select(to_upper("txt"))
```

### Pandas UDF Example

```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

@pandas_udf("double")
def plus_one(batch: pd.Series) -> pd.Series:
    return batch + 1.0

spark.range(0,5).withColumn("x1", plus_one(F.col("id"))).show()
```

## Production Considerations

- Prefer built-ins: They leverage Catalyst and code generation.
- Pandas UDFs: Use for vectorizable heavy operations; ensure Arrow enabled.
- Avoid Python UDF in tight loops; may degrade performance.

## Troubleshooting

- Serialization errors: Inspect function signature and return types.
- Performance regression: Replace Python UDF with native expressions.
- Arrow disabled: Set `spark.conf.set("spark.sql.execution.arrow.pyspark.enabled", "true")`.

## Sample Questions

1. Why are built-in functions preferred?  
2. When use Pandas UDF over scalar UDF?  
3. What overhead does a Python UDF introduce?  
4. How enable Arrow acceleration?  
5. Difference between UDF and UDAF?

## Answers

1. They integrate with Catalyst for optimized execution.  
2. For vectorized operations on batches for performance.  
3. Serialization/deserialization and crossing language boundary.  
4. Set Arrow config to true; ensure environment compatibility.  
5. UDF transforms row-wise; UDAF performs aggregation with state.

## References

- [Spark SQL Functions](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html)
- [Pandas UDFs](https://spark.apache.org/docs/latest/sql-pyspark-pandas-with-arrow.html)

---

Previous: [Performance Optimization: Partitioning, Caching, Broadcast](./07-performance-optimization-caching-partitioning-broadcast.md)  
Next: [Spark SQL and Temp Views](./09-spark-sql-and-temp-views.md)
