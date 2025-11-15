# User-Defined Functions (UDFs)

## Overview

Create and invoke UDFs with/without stateful operators (StateStores for streaming).

## Scalar UDF

```python
from pyspark.sql.types import StringType
from pyspark.sql import functions as F

@F.udf(returnType=StringType())
def to_upper(text: str) -> str:
    return text.upper() if text else None

df_transformed = df.withColumn("name_upper", to_upper(F.col("name")))
```

## Pandas UDF (Vectorized)

```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

@pandas_udf("double")
def multiply_by_two(values: pd.Series) -> pd.Series:
    return values * 2

df_result = df.withColumn("doubled", multiply_by_two(F.col("amount")))
```

## Stateful UDF (Streaming with StateStore)

```python
# Advanced: mapGroupsWithState for streaming stateful operations
from pyspark.sql.streaming import GroupState

def update_state(key, values, state: GroupState):
    # Custom stateful logic
    if state.exists:
        current_count = state.get()
    else:
        current_count = 0
    new_count = current_count + len(values)
    state.update(new_count)
    return (key, new_count)

# Apply in streaming query
stream = input_stream.groupByKey(lambda x: x.key).mapGroupsWithState(update_state)
```

## Best Practices

- Prefer built-in functions over UDFs for performance.  
- Use Pandas UDF for vectorized operations (faster than scalar UDF).  
- Enable Arrow for Pandas UDF acceleration: `spark.conf.set("spark.sql.execution.arrow.pyspark.enabled", "true")`.

## Sample Questions

1. Difference between scalar UDF and Pandas UDF?  
2. Why prefer built-in functions?  
3. When use stateful operations?  
4. How enable Arrow for Pandas UDF?  
5. Performance cost of Python UDFs?

## Answers

1. Scalar processes row-by-row; Pandas UDF processes batches (vectorized).  
2. They integrate with Catalyst for optimizations.  
3. For streaming aggregations requiring state across micro-batches.  
4. Set Arrow config to `true`.  
5. Serialization overhead and no Catalyst optimization.

## References

- [UDFs](https://spark.apache.org/docs/latest/sql-pyspark-pandas-with-arrow.html#pandas-udfs-aka-vectorized-udfs)

---

Previous: [DataFrame Operations](./07-dataframe-operations.md)  
Next: [Broadcast Variables and Accumulators](./09-broadcast-accumulators.md)
