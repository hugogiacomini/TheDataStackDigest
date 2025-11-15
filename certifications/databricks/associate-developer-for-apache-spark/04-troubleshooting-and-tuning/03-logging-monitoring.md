# Logging and Monitoring: Driver/Executor Logs, OOM, Underutilization

## Overview

Diagnose failures using driver/executor logs; identify OOM and underutilization patterns.

## Driver Logs

- Location: Spark UI → Application UI → Driver stdout/stderr.  
- Contains: Application-level errors, configuration, DAG scheduling.

```bash
# Access driver logs in local mode
tail -f $SPARK_HOME/work/driver/stdout
```

## Executor Logs

- Location: Spark UI → Executors tab → Stdout/Stderr links.  
- Contains: Task-level errors, data processing issues, GC logs.

## Diagnosing OOM

### OOM Symptoms

- `java.lang.OutOfMemoryError` in executor logs.  
- Tasks failing repeatedly on same partition.

### OOM Solutions

```python
# Increase executor memory
spark.conf.set("spark.executor.memory", "8g")
spark.conf.set("spark.memory.fraction", "0.8")

# Increase partitions to reduce per-task data
df = df.repartition(500)

# Enable off-heap memory for caching
spark.conf.set("spark.memory.offHeap.enabled", "true")
spark.conf.set("spark.memory.offHeap.size", "4g")
```

## Diagnosing Underutilization

### Underutilization Symptoms

- Executors idle in Spark UI timeline.  
- Low CPU usage in monitoring dashboards.

### Underutilization Solutions

```python
# Increase parallelism
df = df.repartition(200)

# Reduce executor resources to fit more executors
spark.conf.set("spark.executor.cores", "4")
spark.conf.set("spark.executor.memory", "4g")
```

## Best Practices

- Monitor Spark UI during job execution.  
- Correlate OOM errors with data skew and partition sizes.  
- Use dynamic allocation for elastic resource usage.

## Sample Questions

1. Where find executor logs?  
2. Symptom of executor OOM?  
3. How increase executor memory?  
4. Cause of executor underutilization?  
5. Why enable dynamic allocation?

## Answers

1. Spark UI → Executors tab → Stdout/Stderr.  
2. `OutOfMemoryError`, repeated task failures on same partition.  
3. Set `spark.executor.memory`.  
4. Too few partitions or low parallelism.  
5. Auto-scales executors based on workload demand.

## References

- [Monitoring](https://spark.apache.org/docs/latest/monitoring.html)

---

Previous: [AQE](./02-adaptive-query-execution.md)  
Next: [Structured Streaming Engine](../05-structured-streaming/01-streaming-engine-explanation.md)
