# Broadcast Joins: Purpose and Implementation

## Overview

Implement broadcast joins to eliminate shuffle for small dimension tables.

## Concept

- Small table sent to all executors; large table streamed without shuffle.  
- Reduces network and improves performance for star schema joins.

## Implementation

```python
from pyspark.sql.functions import broadcast

# Automatic broadcast (if under threshold)
result = large_facts.join(small_dim, "dimension_id")

# Explicit broadcast hint
result_hinted = large_facts.join(broadcast(small_dim), "dimension_id")

# Check plan for BroadcastHashJoin
result_hinted.explain()
```

## Configuration

```python
# Adjust auto-broadcast threshold (default 10MB)
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "20MB")
```

## Best Practices

- Broadcast tables < 10-50MB.  
- Use explicit `broadcast()` hint when optimizer doesn't detect automatically.  
- Monitor executor memory; large broadcasts can cause OOM.

## Sample Questions

1. When is broadcast join beneficial?  
2. How force broadcast join?  
3. Default auto-broadcast threshold?  
4. What plan operator indicates broadcast?  
5. Risk of broadcasting large tables?

## Answers

1. When one side is small (<threshold) and fits in executor memory.  
2. Use `broadcast(df)` function.  
3. 10MB in vanilla Spark (may vary by platform).  
4. `BroadcastHashJoin` in physical plan.  
5. Executor OOM and driver serialization overhead.

## References

- [Broadcast Joins](https://spark.apache.org/docs/latest/sql-performance-tuning.html#broadcast-hint)

---

Previous: [Broadcast Variables & Accumulators](./09-broadcast-accumulators.md)  
Next: [Performance Tuning](../04-troubleshooting-and-tuning/01-performance-tuning-strategies.md)
