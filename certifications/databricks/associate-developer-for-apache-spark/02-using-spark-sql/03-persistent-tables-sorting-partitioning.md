# Persistent Tables with Sorting and Partitioning

## Overview

Save data to managed/external tables applying sorting, partitioning to optimize retrieval.

## Table Types

- Managed: Spark controls metadata & data; dropped together.  
- External: Metadata in Spark; data persists independently.

## Create Sorted & Partitioned Table

```python
# Write with sorting & partitioning
(df.orderBy("order_date")
   .write
   .mode("overwrite")
   .partitionBy("year", "month")
   .sortBy("order_date")
   .format("delta")
   .saveAsTable("prod.sales.orders_partitioned"))
```

```sql
-- SQL CREATE TABLE with partitioning
CREATE TABLE prod.sales.orders_partitioned
USING DELTA
PARTITIONED BY (year, month)
AS SELECT * FROM source_orders ORDER BY order_date;
```

## Partition Optimization

- Prune partitions during read: `WHERE year = 2025 AND month = 11`.  
- Avoid over-partitioning (thousands of small partitions).  
- Monitor file sizes; compact with `OPTIMIZE` (Delta).

## Z-ORDER Clustering (Delta-specific)

```sql
OPTIMIZE prod.sales.orders_partitioned ZORDER BY (customer_id, region);
```

## Best Practices

- Partition by time (date/year/month) for time-series data.  
- Sort by frequently filtered columns.  
- Use Delta for production tables needing compaction and VACUUM.

## Sample Questions

1. Difference between managed and external tables?  
2. Why partition by date columns?  
3. What does Z-ORDER accomplish?  
4. How enforce sorting in Delta writes?  
5. Risk of excessive partitions?

## Answers

1. Managed deletes data with table; external keeps data independent.  
2. Enables partition pruning for time-range queries.  
3. Co-locates related rows within files for faster multi-column filters.  
4. Use `sortBy` or post-write `OPTIMIZE ZORDER`.  
5. Small file proliferation and metadata overhead.

## References

- [Delta Lake Optimize](https://docs.delta.io/latest/optimizations-oss.html)
- [Partitioning Best Practices](https://spark.apache.org/docs/latest/sql-performance-tuning.html#partitioning)

---

Previous: [SQL Queries on Files](./02-sql-queries-on-files-save-modes.md)  
Next: [Register DataFrames as Temp Views](./04-register-temp-views.md)
