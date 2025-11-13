# dbt Analytics Engineer Certification — Study Guide

This study guide covers all exam topics for the dbt Analytics Engineer Certification. It follows a hands-on, production-minded approach with runnable examples, best practices, and exam-style questions.

- Official exam page: [dbt Analytics Engineer Certification](https://www.getdbt.com/certifications/analytics-engineer-certification-exam)
- Prerequisites: Familiarity with SQL, Git, dbt Core/Cloud basics, and a modern data warehouse (Snowflake/BigQuery/Redshift).

## How to Use This Guide

- Start with Topic 1 and proceed in order; each topic builds on the previous.
- In each topic, look for:
  - Key concepts and definitions
  - Practical examples with code (SQL/YAML/Python/Bash)
  - Best practices and anti-patterns
  - Real-world scenarios and solutions
  - Sample exam-style questions with explanations

## Topic Index

1. Developing dbt models — [01-developing-dbt-models.md](./01-developing-dbt-models.md)
2. Model governance — [02-model-governance.md](./02-model-governance.md)
3. Debugging data modeling errors — [03-debugging-modeling-errors.md](./03-debugging-modeling-errors.md)
4. Managing data pipelines — [04-managing-data-pipelines.md](./04-managing-data-pipelines.md)
5. Implementing dbt tests — [05-implementing-dbt-tests.md](./05-implementing-dbt-tests.md)
6. Documentation — [06-documentation.md](./06-documentation.md)
7. External dependencies — [07-external-dependencies.md](./07-external-dependencies.md)
8. Leveraging state — [08-leveraging-state.md](./08-leveraging-state.md)

## Environment Assumptions

- dbt version: latest stable (note changes for v1.x where relevant)
- Warehouse: Snowflake or BigQuery (examples annotated for dialect specifics)
- Git: GitHub workflow with feature branches + PRs
- Optional: dbt Cloud for jobs, docs, and notifications

## Exam Readiness Checklist

- You can design modular DAGs with DRY macros and materializations
- You can enforce model contracts, access levels, and versioning
- You can debug compilation/runtime errors and read logs effectively
- You can implement robust test suites and integrate in CI
- You can maintain documentation, exposures, and freshness
- You can leverage state and selectors for fast, safe deployments

Good luck on your certification! Proceed to [Topic 1](./01-developing-dbt-models.md).
