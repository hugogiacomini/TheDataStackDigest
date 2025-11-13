# 01 — Developing dbt Models

## Key Concepts and Definitions

- dbt model: A SQL or Python file that transforms data in your warehouse.
- Materializations: `table`, `view`, `incremental`, `ephemeral`, `materialized='incremental'` with `unique_key`.
- DRY and modularity: Use `ref()`, `source()`, CTEs, and macros to avoid repetition.
- DAG: Directed acyclic graph built from `ref()` dependencies.
- Commands: `dbt run`, `dbt test`, `dbt seed`, `dbt docs generate/serve`.
- Sources and seeds: Declare upstream tables as `source()`, load CSVs via `seeds/`.
- Python models: Use `pandas` or PySpark (Spark adapter) to create models alongside SQL.
- Grants: Configure access to relations with `grants` in model configs.

## Practical Examples with Code

### 1. Project Configs and Sources

```yaml
# dbt_project.yml
name: my_project
version: 1.0.0
profile: my_profile

models:
  my_project:
    +materialized: view
    marts:
      +materialized: table
      +grants:
        select: [analyst_role]
```

```yaml
# models/sources.yml
version: 2
sources:
  - name: raw
    schema: raw
    tables:
      - name: orders
      - name: customers
```

### 2. Staging Models: DRY, Naming, and Tests

```sql
-- models/staging/stg_orders.sql
{{ config(materialized='view') }}

with raw as (
  select * from {{ source('raw', 'orders') }}
)

select
  order_id,
  customer_id,
  cast(order_date as date) as order_date,
  cast(total_amount as numeric(18,2)) as total_amount
from raw
where order_status != 'cancelled'
```

```yaml
# models/staging/stg_orders.yml
version: 2
models:
  - name: stg_orders
    description: Cleaned orders staging model
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns: [order_id]
    columns:
      - name: order_id
        tests: [not_null, unique]
      - name: customer_id
        tests: [not_null]
```

### 3. Incremental Fact Model

```sql
-- models/marts/fct_orders.sql
{{
  config(
    materialized='incremental',
    unique_key='order_id',
    on_schema_change='append_new_columns'
  )
}}

with base as (
  select * from {{ ref('stg_orders') }}
)

select *
from base
{% if is_incremental() %}
where order_date >= (select coalesce(max(order_date), '1900-01-01') from {{ this }})
{% endif %}
```

### 4. Macros for Reuse

```sql
-- macros/parse_iso_date.sql
{% macro parse_iso_date(col) %}
    try_cast({{ col }} as date)
{% endmacro %}
```

```sql
-- models/staging/stg_customers.sql
with raw as (
  select * from {{ source('raw', 'customers') }}
)

select
  customer_id,
  customer_name,
  {{ parse_iso_date('created_at') }} as created_at
from raw
```

### 5. Python Model (Optional)

```python
# models/features/customer_features.py
import pandas as pd

def model(dbt, session):
    customers = dbt.ref("stg_customers").to_pandas()
    orders = dbt.ref("stg_orders").to_pandas()

    agg = (
        orders.groupby("customer_id")["total_amount"]
        .agg(["count", "sum"]).reset_index()
        .rename(columns={"count": "order_count", "sum": "total_spend"})
    )

    out = customers.merge(agg, on="customer_id", how="left").fillna({
        "order_count": 0,
        "total_spend": 0.0
    })

    return out
```

### 6. Grants Configuration

```yaml
# models/marts/schema.yml
version: 2
models:
  - name: fct_orders
    config:
      grants:
        select: [analyst_role, finance_reader]
```

## Best Practices and Anti-Patterns

- Prefer `ref()` and `source()` over hardcoded schema.table references.
- Keep staging models 1:1 with sources; apply type casting and basic cleaning.
- Use incremental models for large facts; include `unique_key` and an incremental filter.
- Centralize reusable logic in macros; document macro parameters.
- Avoid overusing `ephemeral` on large joins; can cause heavy compilation queries.
- Do not duplicate business logic across multiple models; DRY via macros/CTEs.
- Document columns and add tests for primary keys and critical assumptions.

## Real-World Scenarios and Solutions

- Need faster daily builds: Convert heavy full-refresh models to incremental with watermark filter.
- Upstream schema changed: Use `on_schema_change='append_new_columns'` and add tests to detect drift.
- Too many repeated casts: Create a macro for standard casts and apply consistently.
- Model access control: Apply `grants` at model/group level; validate with a read-only role.

## Sample Exam Questions

1. Which function builds DAG dependencies in dbt?

- A: `join()`
- B: `graph()`
- C: `ref()`
- D: `link()`

Answer: C. `ref()` defines dependencies and resolves relation names.

1. What’s the main benefit of incremental materialization?

- A: Smaller compiled SQL
- B: Faster development
- C: Process only new/changed data
- D: No warehouse usage

Answer: C. Incremental processes only new/changed rows for performance.

1. Where should you centralize reusable SQL logic?

- A: In each model
- B: In macros
- C: In seeds
- D: In tests

Answer: B. Macros enable DRY, reusable logic.

1. When should you use `source()`?

- A: Referencing upstream warehouse tables
- B: Referencing dbt models
- C: Referencing seeds
- D: Referencing exposures

Answer: A. `source()` references declared external/raw tables.

1. Which config grants read access to a model?

- A: `access`
- B: `grants`
- C: `persist_docs`
- D: `post-hook`

Answer: B. `grants` controls relation-level privileges.

[Back to README](./README.md) • Next: [02 — Model Governance](./02-model-governance.md)
