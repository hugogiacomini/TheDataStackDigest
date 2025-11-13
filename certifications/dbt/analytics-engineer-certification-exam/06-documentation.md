# 06 — Creating and Maintaining dbt Documentation

## Key Concepts and Definitions

- Documentation: Model/source/column descriptions and auto-generated site via `dbt docs`.
- Exposures: Declare downstream consumers (dashboards, ML jobs) with ownership and SLAs.
- Lineage: Visual graph and relationships; use macros to surface lineage where useful.
- Persist docs: Store descriptions in warehouse for discoverability.

## Practical Examples with Code

### 1. Descriptions in YAML

```yaml
# models/marts/schema.yml
version: 2
models:
  - name: fct_orders
    description: Fact table with one row per order
    columns:
      - name: order_id
        description: Primary key for orders
      - name: order_status
        description: Status of the order
```

### 2. Docs Generation and Serving

```bash
 dbt docs generate
 dbt docs serve  # local web server
```

### 3. Persist Docs to Warehouse

```yaml
# dbt_project.yml
name: my_project
version: 1.0.0

models:
  my_project:
    +persist_docs:
      relation: true
      columns: true
```

### 4. Exposures

```yaml
# models/exposures.yml
version: 2
exposures:
  - name: revenue_dashboard
    type: dashboard
    maturity: high
    url: https://bi.company.com/dashboards/revenue
    description: Executive revenue dashboard
    owner:
      name: Finance BI
      email: finance-bi@company.com
    depends_on:
      - ref('fct_revenue')
      - ref('dim_customers')
```

### 5. Macro to Surface Lineage Hints (Optional)

```sql
-- macros/lineage_hint.sql
{% macro lineage_hint(model_name) %}
-- This model depends on {{ model_name }} and feeds executive dashboards
{% endmacro %}
```

```sql
-- models/marts/fct_revenue.sql
{{ lineage_hint('fct_orders') }}
select * from {{ ref('fct_orders') }}
```

## Best Practices and Anti-Patterns

- Keep descriptions concise, clear, and business-facing.
- Maintain exposure ownership and SLA notes; ensure contact is routable.
- Persist docs for better discoverability in BI tools (where supported).
- Avoid stale docs: update descriptions when columns/logic change (enforce via PR checks if possible).

## Real-World Scenarios and Solutions

- Missing context for a field: Add column description and example usage; link to upstream.
- Orphaned dashboard: Create exposure, set owner, and link dependencies for impact analysis.
- Docs drift: Add checklist to PR template to update docs with schema changes.

## Sample Exam Questions

1. What command builds the static documentation site?

- A: `dbt build`
- B: `dbt docs generate`
- C: `dbt run`
- D: `dbt test`

Answer: B. `dbt docs generate` builds the site.

1. Where should model and column descriptions live?

- A: dbt_project.yml
- B: YAML files (schema.yml)
- C: profiles.yml
- D: packages.yml

Answer: B. Put descriptions alongside models in YAML.

1. What is an exposure?

- A: A macro
- B: A downstream consumer declaration
- C: A test
- D: A seed

Answer: B. Exposures declare dashboards/jobs that use models.

1. How do you include descriptions in warehouse metadata for tables?

- A: `persist_docs`
- B: `access`
- C: `grants`
- D: `post-hook`

Answer: A. `persist_docs` writes descriptions to warehouse.

1. Which object links a dashboard to its dependent models for lineage?

- A: seed
- B: exposure
- C: source
- D: snapshot

Answer: B. Exposures link consumers to models.

[Back to README](./README.md) • Prev: [05 — Implementing Tests](./05-implementing-dbt-tests.md) • Next: [07 — External Dependencies](./07-external-dependencies.md)
