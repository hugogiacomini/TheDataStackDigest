# 03 — Debugging Data Modeling Errors

## Key Concepts and Definitions

- Compilation vs runtime errors: Template/Jinja/render issues vs warehouse execution failures.
- Logs and artifacts: `target/` (compiled SQL, `run_results.json`, `manifest.json`).
- Common failure modes: missing `ref()`/`source()`, schema drift, permission errors, SQL syntax.
- Distinguish dbt Core errors from warehouse errors by message origin and stack traces.

## Practical Examples with Code

### 1. Using Compiled Code for Troubleshooting

```bash
# Compile without running
 dbt compile

# Inspect compiled SQL for a failing model
 cat target/compiled/my_project/models/marts/fct_orders.sql
```

### 2. Typical Error Patterns and Fixes

```sql
-- Problem: referenced model not found
select * from {{ ref('stg_order') }}  -- typo
```

```sql
-- Fix: correct ref name
select * from {{ ref('stg_orders') }}
```

```sql
-- Problem: column rename upstream breaks contract
select order_id, amount from {{ ref('stg_orders') }}  -- 'amount' no longer exists
```

```sql
-- Fix: adjust to the new name and update contract/tests
select order_id, total_amount from {{ ref('stg_orders') }}
```

### 3. YAML Compilation Errors

```yaml
# Problem: indentation error in schema.yml
version: 2
models:
 - name: stg_orders   # wrong indent
   columns:
    - name: order_id
      tests: [unique, not_null]
```

```yaml
# Fix: proper indentation
version: 2
models:
  - name: stg_orders
    columns:
      - name: order_id
        tests: [unique, not_null]
```

### 4. Distinguish dbt Core vs Warehouse Errors

- dbt Core: Jinja render failures, selector syntax errors, missing refs/sources.
- Warehouse: SQL compilation at engine, permission denied, relation not found at runtime.

```bash
# Helpful commands
 dbt debug               # environment connectivity
 dbt run -s fct_orders   # run specific model
 dbt test -s stg_orders  # run tests for a model
```

## Best Practices and Anti-Patterns

- Validate YAML with linters and ensure proper indentation.
- Inspect compiled SQL before debugging warehouse errors.
- Keep schema and data-type casts in staging; handle changes via contracts and tests.
- Use selectors to run only failing scopes during triage.
- Avoid hardcoding schemas/tables; always `ref()`/`source()`.

## Real-World Scenarios and Solutions

- Permission denied on production: Ensure service role has `usage/select` on schemas; codify grants in model configs.
- Relation not found: Confirm upstream model run order; add missing dependency via `ref()`.
- Sudden runtime failures: Inspect recent upstream changes (git diff), check contracts/tests, and compiled SQL.

## Sample Exam Questions

1. You see a Jinja rendering error before any SQL hits the warehouse. What type of error is this?

- A: Runtime error
- B: Compilation error
- C: Network error
- D: Permission error

Answer: B. Jinja rendering errors occur during compilation.

1. Where can you find compiled SQL to aid debugging?

- A: `logs/`
- B: `target/compiled/`
- C: `target/run_results.json`
- D: `manifest.json`

Answer: B. Compiled SQL lives under `target/compiled`.

1. A model fails because an upstream column was renamed. What prevents silent breakage for consumers?

- A: Seeds
- B: Contracts and column tests
- C: Ephemeral models
- D: Docs

Answer: B. Contracts/tests detect schema changes.

1. Which command verifies connectivity and profiles configuration?

- A: `dbt docs generate`
- B: `dbt compile`
- C: `dbt debug`
- D: `dbt deps`

Answer: C. `dbt debug` checks environment/connectivity.

1. A warehouse error message mentions permission denied. Which layer is failing?

- A: dbt Core compilation
- B: Warehouse execution
- C: Git
- D: CI tool

Answer: B. Permission errors come from the warehouse.

[Back to README](./README.md) • Prev: [02 — Model Governance](./02-model-governance.md) • Next: [04 — Managing Pipelines](./04-managing-data-pipelines.md)
