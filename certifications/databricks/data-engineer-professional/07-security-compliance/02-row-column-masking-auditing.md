# Row and Column Masking, Auditing, and Advanced Security

## Overview

Implement fine-grained security with Unity Catalog: dynamic views, row filters, column masks, and comprehensive audit trails.

## Prerequisites

- Unity Catalog permissions model
- SQL function and view creation

## Concepts

- Row-level security: Filter rows dynamically based on user identity
- Column masking: Redact or transform sensitive columns per user/group
- Audit logs: Track all data access via `system.access.audit`
- Credential passthrough: Secure access to cloud storage with user identity

## Hands-on Walkthrough

### Row-Level Security (SQL)

```sql
CREATE OR REPLACE VIEW prod.analytics.orders_filtered AS
SELECT * FROM prod.retail.orders_silver
WHERE CASE
  WHEN is_account_group_member('admin') THEN TRUE
  WHEN is_account_group_member('regional_manager') THEN country = current_user_region()
  ELSE customer_id = current_user()
END;
```

### Column Masking Function (SQL)

```sql
CREATE OR REPLACE FUNCTION prod.sec.mask_email(email STRING)
RETURNS STRING
RETURN CASE
  WHEN is_account_group_member('compliance_team') THEN email
  ELSE CONCAT(SUBSTRING(email, 1, 3), '***@***.com')
END;

CREATE OR REPLACE VIEW prod.analytics.customers AS
SELECT customer_id,
       prod.sec.mask_email(email) AS email,
       country
FROM prod.dw.dim_customer;
```

### Apply Mask at Table Level (SQL)

```sql
ALTER TABLE prod.dw.dim_customer
ALTER COLUMN ssn SET MASK prod.sec.mask_ssn;
-- All queries automatically apply mask based on caller
```

### Audit Access (SQL)

```sql
SELECT event_time, user_identity.email, request_params.full_name_arg AS object,
       action_name, response.status_code
FROM system.access.audit
WHERE action_name IN ('getTable', 'readTable', 'createTable')
  AND event_date >= current_date() - INTERVAL 7 DAYS
ORDER BY event_time DESC
LIMIT 200;
```

## Production Considerations

- Performance: Masking functions run per-row; optimize or cache results.
- Consistency: Centralize security logic in reusable functions and views.
- Audit retention: Export audit logs to durable storage for compliance.
- Credential passthrough: Preferred over service principals for user-level accountability.

## Troubleshooting

- Permission denials: Check `GRANT SELECT` on tables and `USE CATALOG/SCHEMA`.
- Mask not applying: Verify function ownership and `ALTER TABLE` success.
- Audit gaps: Ensure Unity Catalog and audit log delivery are enabled.

## Sample Questions

1. How do you implement row-level security in Unity Catalog?  
2. What is the benefit of column masking over denying access?  
3. Where are all data access events logged?  
4. How does credential passthrough improve security?  
5. What is a limitation of dynamic masking functions?

## Answers

1. Use dynamic views with `is_account_group_member()` or `current_user()` filters.  
2. Allows broad access while protecting sensitive data; reduces permission complexity.  
3. `system.access.audit`.  
4. Enforces user-level access to cloud storage; improves accountability.  
5. Per-row overhead; must be optimized for large scans.

## References

- [Row and column security](https://docs.databricks.com/data-governance/unity-catalog/row-and-column-filters.html)
- [Audit logs](https://docs.databricks.com/administration-guide/system-tables/audit-logs.html)

---

Previous: [Security and Compliance](./01-security-compliance.md)
