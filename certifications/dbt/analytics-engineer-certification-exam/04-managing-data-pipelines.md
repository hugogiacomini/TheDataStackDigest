# 04 — Managing Data Pipelines

## Key Concepts and Definitions

- DAG failure management: Identify bottlenecks, retries, concurrency, and dependencies.
- dbt clone: Efficient environment copies using underlying warehouse features (e.g., Snowflake zero-copy clone).
- Integrated tools: Orchestrators/CI and observability systems; map errors back to dbt.
- Selectors/tags: Run subsets for faster recovery; manage ownership and SLAs.

## Practical Examples with Code

### 1. Selector-Driven Recovery

```bash
# Run only models downstream of a changed staging model
 dbt run --select +stg_orders

# Rerun failed-only from last run
 dbt run --select state:failed --state target
```

### 2. Tag Critical Paths and Owners

```yaml
# models/marts/schema.yml
version: 2
models:
  - name: fct_revenue
    description: Revenue fact powering executive dashboards
    tags: [critical, tier1, owner:finance]
```

```bash
# Prioritize critical models
 dbt run --select tag:critical
```

### 3. Using dbt Clone (Adapter-Specific)

```sql
-- Example (Snowflake): Prepare prod-like dev schema using clone
create schema if not exists dev_clone clone prod_schema;
```

```bash
# dbt perspective: switch target schema to the cloned schema in your profile or env
 export DBT_TARGET_SCHEMA=DEV_CLONE
 dbt run -s @marts
```

### 4. Handling Integrated Tool Errors

- Airflow/Orchestrator failure: Check task logs; extract dbt command and re-run locally with same selectors.
- CI failure: Inspect steps; compare artifacts (manifest/run_results) for state differences.
- Observability alert: Map back to run URL, identify failing models, runbook actions.

## Best Practices and Anti-Patterns

- Use tags and selectors to isolate and rerun affected parts quickly.
- Maintain small, well-defined tasks in orchestrators; avoid giant monolithic runs.
- Align environments with reproducible schemas/permissions (consider clone capabilities).
- Keep clear runbooks for critical paths; include common failures and owners.
- Avoid running entire DAGs for small changes; leverage state and selectors.

## Real-World Scenarios and Solutions

- Hourly SLA missed: Rerun `tag:critical` and dependencies first; defer non-critical marts.
- Hotfix needed: Use state selectors to limit blast radius; verify in cloned prod-like schema, then deploy.
- External system outage: Mark upstream sources as delayed; pause dependent tasks; resume with backfill once healthy.

## Sample Exam Questions

1. You need to rerun only impacted models after a small change. What should you use?

- A: Full DAG run
- B: Selectors and tags
- C: Seeds
- D: Exposures

Answer: B. Selectors/tags target specific parts of the graph.

1. What’s a safe approach to test near-production changes?

- A: Run in production directly
- B: Clone prod schema and use it as dev target
- C: Use ephemeral models
- D: Disable tests

Answer: B. Clone-based environments provide realistic validation.

1. An orchestrator task failed on `fct_revenue`. First step?

- A: Edit SQL without review
- B: Check task logs and rerun locally using same selector
- C: Remove the model from DAG
- D: Disable tests

Answer: B. Inspect logs and reproduce locally for root-cause analysis.

1. Which label helps prioritize execution for business-critical models?

- A: owner:marketing
- B: tag:critical
- C: latest_version
- D: access:public

Answer: B. `tag:critical` aids targeted execution.

1. How do you reduce total runtime on small changes?

- A: Always full-refresh
- B: Use state selectors and incremental models
- C: Increase warehouse size indefinitely
- D: Disable tests

Answer: B. State selectors and incrementals minimize work.

[Back to README](./README.md) • Prev: [03 — Debugging Errors](./03-debugging-modeling-errors.md) • Next: [05 — Implementing Tests](./05-implementing-dbt-tests.md)
