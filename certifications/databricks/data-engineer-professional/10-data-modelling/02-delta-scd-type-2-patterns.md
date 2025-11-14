# Delta SCD Type 2 Patterns

## Overview

Implement Slowly Changing Dimension (SCD) Type 2 with Delta Lake, handling late-arriving data, effective dating, and surrogate keys with idempotent MERGEs.

## Prerequisites

- Unity Catalog Delta tables for dimensions and staging
- Familiarity with `MERGE INTO` and window functions

## Concepts

- Effective dating: `effective_from`, `effective_to`, `is_current`
- Surrogate keys: Stable integer keys independent of natural keys
- Late arrivals: Adjust periods while maintaining history integrity

## Hands-on Walkthrough

### Table Design (SQL)

```sql
CREATE TABLE IF NOT EXISTS prod.dw.dim_customer (
  customer_key BIGINT GENERATED ALWAYS AS IDENTITY,
  customer_id STRING,
  email STRING,
  country STRING,
  effective_from TIMESTAMP,
  effective_to TIMESTAMP,
  is_current BOOLEAN
) TBLPROPERTIES (delta.enableChangeDataFeed = true);
```

### SCD Type 2 Upsert (SQL)

```sql
-- Stage incoming changes
CREATE OR REPLACE TEMP VIEW stg_customer AS
SELECT customer_id, email, country, event_ts AS change_ts
FROM prod.staging.customers_changes;

-- Close current rows where values change
MERGE INTO prod.dw.dim_customer AS d
USING (
  SELECT s.* FROM stg_customer s
) AS s
ON d.customer_id = s.customer_id AND d.is_current = true
WHEN MATCHED AND (d.email <> s.email OR d.country <> s.country) THEN
  UPDATE SET d.effective_to = s.change_ts, d.is_current = false;

-- Insert new current rows
INSERT INTO prod.dw.dim_customer
SELECT NULL AS customer_key, s.customer_id, s.email, s.country,
       s.change_ts AS effective_from, TIMESTAMP'9999-12-31' AS effective_to, true AS is_current
FROM stg_customer s;
```

### Late-Arriving Changes (SQL)

```sql
-- If a late change arrives for t_late, close the period and insert a backdated row
MERGE INTO prod.dw.dim_customer d
USING (
  SELECT customer_id, email, country, change_ts FROM stg_customer
) s
ON d.customer_id = s.customer_id AND d.is_current = true AND d.effective_from <= s.change_ts
WHEN MATCHED AND (d.email <> s.email OR d.country <> s.country) THEN
  UPDATE SET d.effective_to = s.change_ts, d.is_current = false;

INSERT INTO prod.dw.dim_customer
SELECT NULL, s.customer_id, s.email, s.country,
       s.change_ts, TIMESTAMP'9999-12-31', true
FROM stg_customer s;
```

## Production Considerations

- Use `IDENTITY` for surrogate keys; expose both surrogate and natural keys.
- Maintain a unique constraint on `(customer_id, effective_from)` logically via MERGE.
- CDF enabled dimensions simplify downstream incrementals.
- Enforce `NOT NULL` on timeline columns; default `effective_to` far future.

## Troubleshooting

- Overlapping periods: Ensure closing updates precede inserts in a transaction.
- Duplicates: Deduplicate staging by `(customer_id, change_ts)`.
- Late-arrival reorder: Use watermarking, then a corrective MERGE.

## Sample Questions

1. Which columns are required to model SCD Type 2 timelines?  
2. How do you prevent overlapping effective periods?  
3. Why are surrogate keys recommended?  
4. How should late-arriving updates be handled?  
5. What Delta feature helps incremental downstream loads?

## Answers

1. `effective_from`, `effective_to`, `is_current`.  
2. Close current rows first, then insert new versions atomically.  
3. They provide stable joins independent of changing natural keys.  
4. Backdate by closing current row at arrival timestamp and inserting new current row.  
5. Change Data Feed (CDF).

## References

- [MERGE INTO syntax](https://docs.databricks.com/sql/language-manual/delta-merge-into.html)
- [Change Data Feed](https://docs.databricks.com/delta/change-data-feed.html)

---

Previous: [Advanced Medallion Architecture](./01-medallion-architecture-advanced.md)  
Next: [Semantic Layer and BI Integration](./03-semantic-layer-and-bi-integration.md)
