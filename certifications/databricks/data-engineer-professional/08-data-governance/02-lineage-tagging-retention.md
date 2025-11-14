# Lineage, Tagging, and Retention Policies

## Overview

Manage metadata, lineage, and lifecycle: Unity Catalog tags, data classification, retention policies, and lineage for impact analysis.

## Prerequisites

- Unity Catalog metastore and admin access
- Familiarity with Delta Lake table properties

## Concepts

- Lineage: Table and column-level dependencies for change impact analysis
- Tags: Key-value metadata for classification (PII, retention, ownership)
- Retention policies: Automated lifecycle management via `VACUUM` and Delta properties
- Classification: PII detection and compliance tagging

## Hands-on Walkthrough

### Apply Tags (SQL)

```sql
ALTER TABLE prod.dw.dim_customer
SET TAGS ('classification' = 'PII', 'owner' = 'data-engineering', 'retention_days' = '2555');

ALTER TABLE prod.dw.dim_customer
ALTER COLUMN ssn SET TAGS ('pii_type' = 'SSN', 'masking' = 'required');
```

### Query Tags (SQL)

```sql
SHOW TAGS ON TABLE prod.dw.dim_customer;
SHOW TAGS ON COLUMN prod.dw.dim_customer.ssn;
```

### Lineage via Unity Catalog API (Python)

```python
import requests

workspace_url = "https://<workspace>.cloud.databricks.com"
token = dbutils.secrets.get(scope="secrets", key="token")

response = requests.get(
    f"{workspace_url}/api/2.0/unity-catalog/lineage-by-table",
    headers={"Authorization": f"Bearer {token}"},
    params={"table_name": "prod.retail.orders_silver"}
)
lineage_data = response.json()
# Inspect upstream and downstream dependencies
```

### Set Retention Policy and VACUUM (SQL)

```sql
ALTER TABLE prod.raw.orders_bronze
SET TBLPROPERTIES (delta.logRetentionDuration = '30 days', delta.deletedFileRetentionDuration = '7 days');

-- Run VACUUM to purge files older than retention
VACUUM prod.raw.orders_bronze RETAIN 168 HOURS;
```

## Production Considerations

- Tag governance: Enforce tagging via policies; audit untagged assets.
- Lineage for CI/CD: Integrate lineage checks before deployments to prevent breakage.
- Retention automation: Schedule VACUUM jobs; balance storage cost vs time-travel needs.
- Classification drift: Periodically scan for new PII columns and tag accordingly.

## Troubleshooting

- Lineage gaps: Ensure SQL/DataFrame ops reference columns explicitly; UDFs may obscure lineage.
- VACUUM failures: Check retention period conflicts or concurrent transactions.
- Tag inconsistency: Centralize tagging in CI/CD pipelines; validate on deployment.

## Sample Questions

1. What are Unity Catalog tags used for?  
2. How do you retrieve table lineage programmatically?  
3. What Delta properties control file retention?  
4. Why is lineage important for schema changes?  
5. How often should VACUUM be run?

## Answers

1. Classification, ownership, retention policies, and compliance metadata.  
2. Unity Catalog REST API (`/api/2.0/unity-catalog/lineage-by-table`).  
3. `delta.logRetentionDuration` and `delta.deletedFileRetentionDuration`.  
4. Identifies downstream dependencies to prevent breaking changes.  
5. Periodically (e.g., weekly) based on retention policy and storage cost tolerance.

## References

- [Unity Catalog tags](https://docs.databricks.com/data-governance/unity-catalog/tags.html)
- [Data lineage](https://docs.databricks.com/data-governance/unity-catalog/data-lineage.html)
- [VACUUM command](https://docs.databricks.com/sql/language-manual/delta-vacuum.html)

---

Previous: [Data Governance](./01-data-governance.md)
