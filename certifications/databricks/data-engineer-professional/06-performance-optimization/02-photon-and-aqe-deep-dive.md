# Photon Engine and AQE Deep Dive

## Overview

Optimize query performance with Photon (vectorized execution) and Adaptive Query Execution (AQE): dynamic optimization, broadcast/shuffle tuning.

## Prerequisites

- Understanding of Spark physical plans
- Cluster configuration and runtime selection

## Concepts

- Photon: Vectorized C++ engine for scan/filter/aggregate/join acceleration
- AQE: Runtime optimizations (coalesce partitions, skew join, dynamic broadcast)
- When Photon applies: Certain operations benefit; others fall back to Spark
- Observability: Spark UI shows Photon vs non-Photon stages

## Hands-on Walkthrough

### Enable Photon (Cluster Config)

```json
{
  "spark_version": "13.3.x-photon-scala2.12",
  "node_type_id": "i3.xlarge",
  "num_workers": 4,
  "runtime_engine": "PHOTON"
}
```

### Enable AQE (Python)

```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.autoBroadcastJoinThreshold", "10MB")
```

### Verify Photon Acceleration (Spark UI)

- Navigate to SQL/DataFrame query details
- Check physical plan: `PhotonScan`, `PhotonHashAggregate` indicate Photon usage
- Compare execution time before/after Photon enablement

### Dynamic Broadcast Join (SQL + AQE)

```sql
-- AQE can promote small shuffle to broadcast at runtime
SELECT o.order_id, c.country
FROM prod.retail.orders_large o
JOIN prod.dw.dim_customer c ON o.customer_id = c.customer_id;
-- AQE detects c is small post-filter and broadcasts
```

## Production Considerations

- Photon licensing: Requires Databricks Runtime with Photon; incurs additional DBU cost.
- Workload suitability: Photon excels at scans, filters, aggregates; less benefit for complex UDFs.
- AQE overhead: Slight planning overhead; net benefit for diverse queries.
- Monitoring: Use Spark UI and query history to quantify speedup and cost trade-offs.

## Troubleshooting

- Photon not applying: Check runtime version, operation compatibility, and fallback to Spark.
- AQE not coalescing: Ensure small partition sizes and `coalescePartitions.enabled = true`.
- Skew join not triggering: Tune `skewedPartitionThresholdInBytes` based on data distribution.

## Sample Questions

1. What is the Photon engine optimized for?  
2. When does AQE decide to broadcast a join?  
3. How do you verify Photon is being used?  
4. What are the trade-offs of enabling Photon?  
5. How does AQE handle post-filter partition size?

## Answers

1. Vectorized execution for scans, filters, aggregates, and joins.  
2. Dynamically when post-shuffle table size is below `autoBroadcastJoinThreshold`.  
3. Check physical plan in Spark UI for `Photon*` operators.  
4. Faster queries but higher DBU costs; not all operations benefit.  
5. Coalesces small partitions dynamically to reduce task overhead.

## References

- [Photon engine](https://docs.databricks.com/runtime/photon.html)
- [Adaptive Query Execution](https://docs.databricks.com/optimizations/aqe.html)

---

Previous: [Performance Optimization](./01-performance-optimization.md)
