# Execute SQL Queries Directly on Files

## Overview

Query ORC, JSON, CSV, Text, Delta files using SQL syntax without explicit DataFrame registration; understand save modes.

## Direct File Queries

```sql
-- Parquet
SELECT * FROM parquet.`/mnt/data/sales/*.parquet` WHERE region = 'EMEA';

-- JSON
SELECT event_type, COUNT(*) FROM json.`/mnt/raw/events/` GROUP BY event_type;

-- CSV
SELECT * FROM csv.`/mnt/data/customers.csv`;

-- Delta
SELECT customer_id, SUM(amount) FROM delta.`/mnt/data/transactions/` GROUP BY customer_id;

-- ORC
SELECT * FROM orc.`/mnt/warehouse/logs/` LIMIT 10;
```

## Save Modes in SQL

```sql
-- Overwrite existing table
CREATE OR REPLACE TABLE prod.sales USING DELTA AS SELECT * FROM source;

-- Insert append
INSERT INTO prod.sales SELECT * FROM new_data;
```

## Example Workflow

```python
# Query file directly
result = spark.sql("SELECT region, SUM(revenue) FROM parquet.`/mnt/sales/` GROUP BY region")

# Save with overwrite mode
result.write.mode("overwrite").format("delta").saveAsTable("prod.regional_revenue")
```

## Best Practices

- Use Delta for production tables requiring updates and time-travel.  
- Leverage partition pruning by referencing partition columns in WHERE.  
- Avoid querying large unfiltered text files; prefer columnar formats.

## Sample Questions

1. Syntax to query Parquet file in SQL?  
2. What save mode appends without replacing?  
3. Difference between CREATE OR REPLACE vs INSERT INTO?  
4. Why prefer Delta for transactional workloads?  
5. What format supports schema-on-read from semi-structured data?

## Answers

1. `SELECT * FROM parquet.\`/path/\``.  
2. `append` mode or `INSERT INTO`.  
3. CREATE OR REPLACE overwrites; INSERT INTO appends.  
4. ACID guarantees, time-travel, schema evolution.  
5. JSON (and Avro with schema registry).

## References

- [Spark SQL File Queries](https://spark.apache.org/docs/latest/sql-data-sources-load-save-functions.html)
- [Delta Lake](https://docs.delta.io/)

---

Previous: [Data Sources & Partitioning](./01-data-sources-read-write-partitioning.md)  
Next: [Persistent Tables with Sorting & Partitioning](./03-persistent-tables-sorting-partitioning.md)
