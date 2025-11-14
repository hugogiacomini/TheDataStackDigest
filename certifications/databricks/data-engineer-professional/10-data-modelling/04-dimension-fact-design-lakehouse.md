# Dimension and Fact Design in the Lakehouse

## Overview

Model robust dimensional schemas on Delta Lake: surrogate keys, role-playing dimensions, degenerate dimensions, and fact table strategies.

## Prerequisites

- Understanding of dimensional modeling terminology
- Delta Lake fundamentals

## Concepts

- Dimension types: Conformed, role-playing, slowly changing (Type 1/2)
- Fact types: Transactional, periodic snapshot, accumulating snapshot
- Keys: Surrogate vs natural
- Grain: Declare grain first; derive dimensions and measures

## Hands-on Walkthrough

### Define Dimensions (SQL)

```sql
CREATE TABLE IF NOT EXISTS prod.dw.dim_date (
  date_key INT, calendar_date DATE, year INT, month INT, day INT,
  PRIMARY KEY (date_key)
);

CREATE TABLE IF NOT EXISTS prod.dw.dim_product (
  product_key BIGINT GENERATED ALWAYS AS IDENTITY,
  product_id STRING,
  category STRING,
  brand STRING,
  effective_from TIMESTAMP,
  effective_to TIMESTAMP,
  is_current BOOLEAN
);
```

### Transaction Fact (SQL)

```sql
CREATE TABLE IF NOT EXISTS prod.dw.fact_orders (
  order_id STRING,
  order_ts TIMESTAMP,
  date_key INT,
  customer_key BIGINT,
  product_key BIGINT,
  quantity INT,
  amount DECIMAL(18,2)
) PARTITIONED BY (date_key);
```

### Populate Fact with Conformed Keys (SQL)

```sql
INSERT INTO prod.dw.fact_orders
SELECT o.order_id,
       o.order_ts,
       d.date_key,
       c.customer_key,
       p.product_key,
       o.quantity,
       o.total_amount
FROM prod.retail.orders_silver o
JOIN prod.dw.dim_date d ON d.calendar_date = date(o.order_ts)
JOIN prod.dw.dim_customer c ON c.customer_id = o.customer_id AND c.is_current = true
JOIN prod.dw.dim_product p ON p.product_id = o.product_id AND p.is_current = true;
```

## Production Considerations

- Grain clarity: Define measures that match the declared grain.
- Surrogate keys: Required for stable joins as natural keys change.
- Partitioning: Use time-based partitions for large facts; Z-ORDER on common filters.
- Snapshots: Use periodic or accumulating snapshots for lifecycle analytics.

## Troubleshooting

- Many-to-many joins: Introduce bridge tables to avoid double counting.
- Late dimensions: Use SCD2 alignment by effective date to pick correct keys.
- Null foreign keys: Create an unknown member row (e.g., `0`) for robustness.

## Sample Questions

1. Why define the fact table grain first?  
2. When should accumulating snapshots be used?  
3. What problem do surrogate keys solve?  
4. How do you handle missing dimension references?

## Answers

1. It drives the choice of measures and required dimensions.  
2. For lifecycle processes with milestones (e.g., order → ship → deliver).  
3. Stable joins independent of mutable natural keys.  
4. Use an unknown member and treat as a valid foreign key.

## References

- [Dimensional modeling on the Lakehouse](https://www.databricks.com/blog)
- [Delta Lake best practices](https://docs.databricks.com/delta/best-practices.html)

---

Previous: [Semantic Layer and BI Integration](./03-semantic-layer-and-bi-integration.md)  
Next: [Incremental Modelling Strategies](./05-incremental-modelling-strategies.md)
