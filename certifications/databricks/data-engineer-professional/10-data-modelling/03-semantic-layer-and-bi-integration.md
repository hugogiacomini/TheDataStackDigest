# Semantic Layer and BI Integration

## Overview

Design a governed semantic layer with Unity Catalog, views/materialized views, and performance features to integrate with BI tools.

## Prerequisites

- Unity Catalog permissions model
- Databricks SQL (DBSQL) and BI connectivity (JDBC/ODBC)

## Concepts

- Semantic layer: Stable, business-friendly entities (metrics, dimensions).
- Views vs materialized views: Freshness vs compute trade-offs.
- Access control: Secure views, row/column masking, tags/classifications.
- Metrics reliability: Consistent definitions and change management.

## Hands-on Walkthrough

### Business View Layer (SQL)

```sql
CREATE OR REPLACE VIEW prod.analytics.orders AS
SELECT o.order_id, o.customer_id, o.order_date, o.total_amount,
       c.country, c.segment
FROM prod.retail.orders_silver o
LEFT JOIN prod.dw.dim_customer c
  ON o.customer_id = c.customer_id AND c.is_current = true;
```

### Metrics via Materialized View (SQL)

```sql
CREATE OR REPLACE MATERIALIZED VIEW prod.analytics.mv_mtd_revenue AS
SELECT date_trunc('month', order_date) AS month,
       SUM(total_amount) AS revenue
FROM prod.analytics.orders
GROUP BY 1;
```

### Masking Policy Example (SQL)

```sql
CREATE OR REPLACE FUNCTION prod.sec.mask_email(email STRING) RETURNS STRING
RETURN CASE WHEN is_account_group_member('analyst') THEN email ELSE '***@***' END;

ALTER VIEW prod.analytics.orders
ALTER COLUMN customer_email SET MASK prod.sec.mask_email;
```

## Production Considerations

- Promotion: Use environment-specific catalogs (dev/test/prod) with Bundles.
- Caching: Leverage DBSQL serverless or warehouse caching for hot metrics.
- Ownership: Assign data product owners; document definitions and SLAs.
- Tagging: Apply PII tags, classification, and retention policies.

## Troubleshooting

- Inconsistent metrics: Centralize definitions; avoid duplicated logic across tools.
- Permission denials: Validate `SELECT` on source tables and view object ACLs.
- Freshness lag: Switch to materialized views or direct tables for strict SLAs.

## Sample Questions

1. When should materialized views be preferred over views?  
2. How can you prevent exposure of PII in the semantic layer?  
3. What is the benefit of centralizing metric definitions?  
4. How do you handle environment promotions safely?

## Answers

1. When aggregations are heavy and require strict, predictable latency.  
2. Use masking policies and column-level permissions; tag PII.  
3. Consistency, auditability, and reduced metric drift across tools.  
4. Promote via Bundles to env-specific catalogs with approvals.

## References

- [Databricks views](https://docs.databricks.com/sql/language-manual/sql-ref-syntax-ddl-create-view.html)
- [Masking policies](https://docs.databricks.com/data-governance/unity-catalog/column-level-security.html)

---

Previous: [Delta SCD Type 2 Patterns](./02-delta-scd-type-2-patterns.md)  
Next: [Dimension and Fact Design](./04-dimension-fact-design-lakehouse.md)
