# Advanced Medallion Architecture for the Lakehouse

## Overview

Deep dive on designing Bronze/Silver/Gold layers for reliability, performance, and governance. Covers schema enforcement, expectations, idempotent loads, change propagation, and serving models for BI.

## Prerequisites

- Unity Catalog enabled workspaces and catalogs
- Delta Lake tables and basic Spark SQL knowledge
- Familiarity with Auto Loader or Lakeflow

## Concepts

- Bronze: Raw ingestion with minimal transforms; preserve lineage and raw fidelity.
- Silver: Cleansed/conformed; apply constraints, dedupe, late-arrival handling.
- Gold: Serving models; dimensional/fact tables, aggregates, and semantic views.
- Contracts: Schemas, column-level constraints, expectations, and governance tags.
- Idempotency: Deterministic reruns using checkpointing + MERGE patterns.

## Hands-on Walkthrough

### Create Catalog, Schema, and Paths (SQL)

```sql
CREATE CATALOG IF NOT EXISTS prod;
USE CATALOG prod;
CREATE SCHEMA IF NOT EXISTS retail;
```

### Bronze Ingestion (Python with Auto Loader)

```python
from pyspark.sql.functions import input_file_name, current_timestamp

raw_df = (
    spark.readStream.format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.inferColumnTypes", "true")
    .load("s3://company-raw/retail/orders/")
    .withColumn("_ingest_file", input_file_name())
    .withColumn("_ingest_ts", current_timestamp())
)

(raw_df.writeStream
 .option("checkpointLocation", "/mnt/checkpoints/retail/orders_bronze")
 .trigger(availableNow=True)
 .toTable("prod.retail.orders_bronze"))
```

### Silver Cleansing + Dedupe (SQL)

```sql
CREATE TABLE IF NOT EXISTS prod.retail.orders_silver AS
SELECT * FROM prod.retail.orders_bronze WHERE 1=0;

MERGE INTO prod.retail.orders_silver AS s
USING (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY _ingest_ts DESC) AS rn
  FROM prod.retail.orders_bronze
) b
ON s.order_id = b.order_id
WHEN MATCHED AND b.rn = 1 THEN UPDATE SET *
WHEN NOT MATCHED AND b.rn = 1 THEN INSERT *;
```

### Gold Serving View + Materialized View (SQL)

```sql
CREATE OR REPLACE VIEW prod.retail.vw_daily_revenue AS
SELECT order_date, SUM(total_amount) AS revenue
FROM prod.retail.orders_silver
GROUP BY order_date;

CREATE OR REPLACE MATERIALIZED VIEW prod.retail.mv_daily_revenue AS
SELECT order_date, SUM(total_amount) AS revenue
FROM prod.retail.orders_silver
GROUP BY order_date;
```

## Production Considerations

- Partitioning and Z-Ordering: Choose keys that reduce shuffle and optimize queries.
- Expectations: Apply on Bronze to quarantine bad records; alert via system tables.
- Backfill Strategy: Run bronze → silver MERGE in chunks; maintain watermarks.
- Lineage: Use Unity Catalog lineage for impact analysis when changing schemas.
- Cost: Prefer views for frequent updates; materialized views when latency SLAs demand.

## Troubleshooting

- Skewed keys: Add salting or repartition before aggregations.
- Late arrivals: Watermarking + MERGE with effective timestamps.
- Duplicate keys: Use window dedupe before silver MERGE.

## Sample Questions

1. Which layer should enforce deduplication and normalization?  
2. When do materialized views outperform standard views?  
3. What makes a bronze load idempotent?  
4. How do you manage schema changes across layers?  
5. When to use Z-ORDER vs partitioning?

## Answers

1. Silver layer.  
2. When repeated aggregations have tight latency SLAs and stable definitions.  
3. Checkpointing + deterministic MERGE/upsert into Delta tables.  
4. Contracts + lineage; evolve bronze first, then apply controlled promotions to silver/gold.  
5. Partition for pruning on high-cardinality filters; Z-ORDER for co-locating related columns within files.

## References

- [Delta Lake documentation](https://docs.databricks.com/delta/)
- [Materialized views in Databricks SQL](https://docs.databricks.com/sql/language-manual/sql-ref-syntax-ddl-create-mview.html)

---

Next: [Delta SCD Type 2 Patterns](./02-delta-scd-type-2-patterns.md)
