# Spark Deployment Modes: Client, Cluster, Local

## Overview

Spark supports three deployment modes determining where driver runs.

## Local Mode

- **Driver & Executors**: Same JVM on single machine.  
- **Use Case**: Testing, development, small datasets.

```python
spark = SparkSession.builder \
    .master("local[4]") \
    .appName("LocalTest") \
    .getOrCreate()
```

## Client Mode

- **Driver**: Runs on client machine (outside cluster).  
- **Executors**: Run on cluster worker nodes.  
- **Use Case**: Interactive notebooks, debugging.

```bash
spark-submit --master yarn --deploy-mode client script.py
```

**Risk**: Driver failure stops application; network latency between driver and executors.

## Cluster Mode

- **Driver**: Runs on cluster (worker node or master).  
- **Executors**: Run on cluster worker nodes.  
- **Use Case**: Production jobs, scheduled pipelines.

```bash
spark-submit --master yarn --deploy-mode cluster script.py
```

**Benefits**: Fault-tolerant (driver restarts on failure); reduced network latency.

## Comparison Table

| Mode    | Driver Location | Use Case            | Fault Tolerance |
|---------|----------------|---------------------|----------------|
| Local   | Local JVM      | Development/Testing | None           |
| Client  | Client Machine | Interactive         | Low            |
| Cluster | Cluster Node   | Production          | High           |

## Best Practices

- Use **local** for rapid development and unit testing.  
- Use **client** for interactive notebooks with small jobs.  
- Use **cluster** for production pipelines and scheduled jobs.

## Sample Questions

1. Where does driver run in cluster mode?  
2. Why prefer cluster mode for production?  
3. When use client mode?  
4. Local mode parallelism syntax?  
5. Risk of client mode?

## Answers

1. On a cluster worker node or master.  
2. Fault-tolerant driver, reduced latency, better resource isolation.  
3. Interactive sessions, debugging with immediate feedback.  
4. `local[N]` where N is thread count (e.g., `local[4]`).  
5. Driver on client machine; failure stops application.

## References

- [Deployment Modes](https://spark.apache.org/docs/latest/cluster-overview.html#cluster-mode-overview)

---

Previous: [Spark Connect Features](./01-spark-connect-features.md)  
Next: [Pandas API on Spark Advantages](../07-using-pandas-api-on-spark/01-pandas-api-advantages.md)
