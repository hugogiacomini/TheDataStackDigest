# Building and Deploying with Databricks Asset Bundles (DABs)

## Overview

Databricks Asset Bundles (DABs) standardize packaging and deploying Databricks resources (jobs, pipelines, permissions, notebooks, files) across environments with a single `databricks.yml`. Bundles integrate with CI/CD to enable repeatable, auditable releases.

## Core Concepts

- `databricks.yml`: declarative definition of resources and environment settings.
- Environments: `dev`, `staging`, `prod` with overrides and variables.
- Commands: `databricks bundle validate|deploy|run`.

## Example databricks.yml

```yaml
bundle:
  name: orders-platform

targets:
  dev:
    workspace:
      host: https://adb-123456.11.azuredatabricks.net
    variables:
      catalog: dev_catalog
  prod:
    workspace:
      host: https://adb-654321.11.azuredatabricks.net
    variables:
      catalog: prod_catalog

resources:
  jobs:
    orders_etl:
      name: orders-etl
      tasks:
        - task_key: ingest
          notebook_task:
            notebook_path: ./notebooks/ingest
            base_parameters:
              run_date: "{{var.run_date}}"
          new_cluster:
            spark_version: 14.3.x-scala2.12
            node_type_id: Standard_DS3_v2
            num_workers: 4
        - task_key: transform
          depends_on: [ingest]
          notebook_task:
            notebook_path: ./notebooks/transform
  pipelines:
    orders_pipeline:
      name: orders-pipeline
      storage: /pipelines/storage/orders
      clusters:
        - label: default
          num_workers: 3
      libraries:
        - notebook: ./notebooks/pipeline
```

## Deploy and Run

```bash
# Validate configuration
databricks bundle validate --target dev

# Deploy resources to dev
databricks bundle deploy --target dev

# Trigger a job in the bundle with parameters
databricks bundle run orders_etl --target dev -- --notebook-params '{"run_date":"2025-11-01"}'
```

## CI/CD Integration (GitHub Actions)

```yaml
name: deploy-orders-bundle
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Databricks CLI
        uses: databricks/setup-cli@v1
      - name: Auth
        run: |
          databricks auth login --host "$DATABRICKS_HOST" --token "$DATABRICKS_TOKEN"
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
      - name: Validate
        run: databricks bundle validate --target prod
      - name: Deploy
        run: databricks bundle deploy --target prod
```

## Best Practices

- Keep resources small and composable; avoid monolithic jobs.
- Externalize environment-specific values with variables and secrets.
- Validate in CI; deploy on protected branches only.

## Mock Questions

1. **Which file defines resources and environment-specific overrides for Bundles?**  
    a. `dbt_project.yml`  
    b. `databricks.yml`  
    c. `settings.json`  
    d. `pom.xml`

2. **Which command deploys resources to a target workspace?**  
    a. `databricks bundle run`  
    b. `databricks bundle deploy`  
    c. `databricks jobs run-now`  
    d. `databricks repos update`

3. **Where should environment-specific workspace hosts be defined?**  
    a. In notebook widgets  
    b. In cluster tags  
    c. Under `targets` in `databricks.yml`  
    d. In `requirements.txt`

## Answers

1. b
2. b
3. c

## References

- [Databricks Asset Bundles (DABs) documentation](https://docs.databricks.com/aws/en/dev-tools/bundles/)
