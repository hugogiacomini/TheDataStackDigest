# 05 — Implementing dbt Tests

## Key Concepts and Definitions

- Test types: generic (schema), singular (custom SQL), custom generic (reusable), source tests.
- Severity and outcomes: `warn` vs `error`; gating policies in CI/production.
- Coverage: Keys, nullability, referential integrity, accepted values, freshness.
- Workflow: Write → run locally → integrate into CI → monitor failures.

## Practical Examples with Code

### 1. Generic Tests on Models

```yaml
# models/marts/schema.yml
version: 2
models:
  - name: fct_orders
    columns:
      - name: order_id
        tests:
          - not_null
          - unique
      - name: customer_id
        tests:
          - not_null
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: order_status
        tests:
          - accepted_values:
              values: ['completed', 'pending', 'returned']
```

### 2. Source Tests and Freshness

```yaml
# models/sources.yml
version: 2
sources:
  - name: raw
    schema: raw
    freshness:
      warn_after: {count: 2, period: hour}
      error_after: {count: 6, period: hour}
    tables:
      - name: orders
        loaded_at_field: _loaded_at
        columns:
          - name: order_id
            tests: [not_null]
```

```bash
# Run freshness checks
 dbt source freshness --select source:raw.orders
```

### 3. Singular and Custom Generic Tests

```sql
-- tests/no_future_order_dates.sql (singular)
select 1
where exists (
  select 1
  from {{ ref('fct_orders') }}
  where order_date > current_date
)
```

```sql
-- macros/tests/no_recent_gaps.sql (custom generic)
{% test no_recent_gaps(model, date_column) %}
select 1
from (
  select {{ date_column }}, lead({{ date_column }}) over (order by {{ date_column }}) as next_dt
  from {{ model }}
) t
where datediff(day, {{ date_column }}, next_dt) > 7
{% endtest %}
```

```yaml
# apply custom generic test
version: 2
models:
  - name: fct_orders
    tests:
      - no_recent_gaps:
          date_column: order_date
```

### 4. Severity and CI Integration

```yaml
# dbt_project.yml
name: my_project
version: 1.0.0

tests:
  +severity: error  # default; can override per test
```

```bash
# CI steps (example)
 dbt deps
 dbt seed --full-refresh
 dbt run --select state:modified+ --defer --state artifacts/prod
 dbt test --select state:modified+ --fail-fast
```

## Best Practices and Anti-Patterns

- Prioritize primary/foreign keys, nullability, and accepted values on critical models.
- Use `relationships` to guarantee referential integrity to dimensions.
- Parameterize custom generic tests; keep singular tests minimal and focused.
- Treat test failures as deploy blockers in prod; use `warn` only for non-critical.
- Avoid over-testing trivial columns; focus on correctness and SLA assurance.

## Real-World Scenarios and Solutions

- Spike in invalid statuses: `accepted_values` fails; fix upstream mapping and add guard in staging.
- Missing customers: `relationships` fails; investigate late-arriving dimensions and adjust load order.
- Data gaps: Implement `no_recent_gaps` and create alert on failure.

## Sample Exam Questions

1. Which test ensures referential integrity between fact and dimension?

- A: unique
- B: relationships
- C: accepted_values
- D: not_null

Answer: B. `relationships` validates foreign key integrity.

1. When should you use a singular test?

- A: For reusable logic
- B: For one-off checks expressed in SQL
- C: For nullability
- D: For accepted values

Answer: B. Singular tests are bespoke SQL checks.

1. How should critical test failures be handled in production?

- A: Ignore
- B: Log only
- C: Fail the deployment
- D: Convert to warnings

Answer: C. Critical failures should block deploys.

1. How do you test that a set of values is constrained?

- A: relationships
- B: accepted_values
- C: not_null
- D: unique

Answer: B. `accepted_values` enforces enumerations.

1. Where do you define default severity for tests?

- A: schema.yml
- B: dbt_project.yml
- C: profiles.yml
- D: packages.yml

Answer: B. Default test severity is in `dbt_project.yml`.

[Back to README](./README.md) • Prev: [04 — Managing Pipelines](./04-managing-data-pipelines.md) • Next: [06 — Documentation](./06-documentation.md)
