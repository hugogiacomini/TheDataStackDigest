# 02 — Model Governance

## Key Concepts and Definitions

- Contracts: Enforce model schemas to prevent breaking changes.
- Access levels: `access: private | protected | public` to control referenceability.
- Versions: Versioned models (v1, v2) with `latest_version` and deprecation.
- Grants: Warehouse-level privileges on relations (e.g., `select`).
- Deprecation: Mark old versions for removal after migration windows.

## Practical Examples with Code

### 1. Enforcing Contracts

```yaml
# models/marts/customers.yml
version: 2
models:
  - name: dim_customers
    description: Canonical customer dimension
    access: public
    config:
      contract:
        enforced: true
    columns:
      - name: customer_id
        data_type: integer
        tests: [not_null, unique]
      - name: customer_name
        data_type: varchar
      - name: email
        data_type: varchar
      - name: created_at
        data_type: timestamp
```

```sql
-- models/marts/dim_customers.sql
{{ config(materialized='table', access='public', contract={'enforced': true}) }}

select
  customer_id,
  customer_name,
  email,
  created_at
from {{ ref('stg_customers') }}
```

### 2. Versioned Models and Deprecation

```yaml
# models/marts/dim_customers_versions.yml
version: 2
models:
  - name: dim_customers
    latest_version: 2
    access: public
    versions:
      - v: 1
        deprecation_date: 2025-06-30
        description: Original schema
      - v: 2
        description: Adds segmentation columns
        columns:
          - name: customer_id
          - name: customer_name
          - name: email
          - name: created_at
          - name: customer_segment
```

```sql
-- models/marts/dim_customers_v1.sql
{{ config(materialized='table', access='public', version=1) }}
select customer_id, customer_name, email, created_at from {{ ref('stg_customers') }}
```

```sql
-- models/marts/dim_customers_v2.sql
{{ config(materialized='table', access='public', version=2) }}
select customer_id, customer_name, email, created_at, customer_segment from {{ ref('int_customer_enrichment') }}
```

### 3. Access Levels

```yaml
# models/access.yml
version: 2
models:
  - name: stg_orders
    access: private  # not referenceable outside package
  - name: int_order_metrics
    access: protected  # referenceable within project only
  - name: dim_customers
    access: public   # referenceable by other projects
```

### 4. Grants for Warehouse Privileges

```yaml
# models/marts/schema.yml
version: 2
models:
  - name: dim_customers
    config:
      grants:
        select: [bi_reader, analyst_role]
```

## Best Practices and Anti-Patterns

- Always enforce contracts on public models; add tests for keys and constraints.
- Prefer additive changes; avoid removing/renaming columns without versioning.
- Maintain clear deprecation timelines; communicate widely and track adoption.
- Keep staging/internal models `private` or `protected` unless intentionally shared.
- Avoid exposing unstable models; stabilize logic and documentation first.

## Real-World Scenarios and Solutions

- Breaking change needed: Introduce v2, keep v1 during migration, communicate and measure adoption, remove v1 after deadline.
- Unintended downstream break: Contracts catch mismatches at build time—roll back or ship v2 instead.
- Access leakage: Audit access settings; move intermediate models back to `private`/`protected`.

## Sample Exam Questions

1. What is the primary purpose of model contracts?

- A: Improve performance
- B: Enforce schema and prevent breaking changes
- C: Reduce storage
- D: Enable exposures

Answer: B. Contracts enforce schema stability for downstream users.

1. Which access level allows other projects to reference a model?

- A: private
- B: protected
- C: public
- D: external

Answer: C. `public` is referenceable across projects.

1. How should you introduce a schema change that removes a column from a widely used model?

- A: Remove the column immediately
- B: Add a macro
- C: Create a new version and deprecate the old one
- D: Change the data type in place

Answer: C. Versioning enables safe migration without breaking consumers.

1. What should be the default access for staging models?

- A: public
- B: protected or private
- C: external
- D: shared

Answer: B. Staging/internal models should not be public by default.

1. What is the best way to grant read access to a model for BI users?

- A: Add a test
- B: Use `grants` config
- C: Use `persist_docs`
- D: Use `post-hook`

Answer: B. `grants` config sets warehouse privileges.

[Back to README](./README.md) • Prev: [01 — Developing dbt Models](./01-developing-dbt-models.md) • Next: [03 — Debugging Errors](./03-debugging-modeling-errors.md)
