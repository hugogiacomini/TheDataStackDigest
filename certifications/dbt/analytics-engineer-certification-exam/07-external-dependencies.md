# 07 — Implementing and Maintaining External Dependencies

## Key Concepts and Definitions

- Exposures: Declare downstream consumers (dashboards, ML, apps) for ownership and lineage.
- Source freshness: Validate data recency with `loaded_at_field` and thresholds.
- Packages: Reuse community or internal macros/models via `packages.yml`.
- Dependency management: Pin versions, review changelogs, update safely.

## Practical Examples with Code

### 1. Exposures for Critical Dashboards

```yaml
# models/exposures.yml
version: 2
exposures:
  - name: exec_kpi_dashboard
    type: dashboard
    maturity: high
    url: https://bi.company.com/dashboards/exec-kpi
    owner:
      name: Exec BI
      email: exec-bi@company.com
    depends_on:
      - ref('fct_revenue')
      - ref('agg_kpis_daily')
```

### 2. Source Freshness

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
```

```bash
 dbt source freshness --select source:raw.orders
```

### 3. Packages and Version Pinning

```yaml
# packages.yml
packages:
  - package: dbt-labs/dbt_utils
    version: ">=1.0.0 <2.0.0"
  - git: "https://github.com/yourorg/internal_dbt_macros.git"
    revision: v0.4.1
```

```bash
 dbt deps  # install/update packages
```

## Best Practices and Anti-Patterns

- Keep exposures up to date with owners and URLs; use them for impact analysis.
- Set realistic freshness thresholds and monitor failures; alert stakeholders.
- Pin package versions; review release notes before upgrades.
- Avoid unpinned git SHAs that move unexpectedly; use tags/releases.

## Real-World Scenarios and Solutions

- Dashboard shows stale data: Freshness check failing—coordinate with ingestion team; temporarily relax SLA if appropriate.
- Package upgrade breaks macro: Pin previous working version; open issue/PR upstream; plan upgrade window.
- Unknown downstream impacts: Leverage exposures to enumerate affected consumers during changes.

## Sample Exam Questions

1. Why define exposures?

- A: To enforce contracts
- B: To declare downstream consumers for lineage and ownership
- C: To run seeds
- D: To manage profiles

Answer: B. Exposures document consumers and enable impact analysis.

1. What command checks data recency for a source table?

- A: `dbt docs generate`
- B: `dbt source freshness`
- C: `dbt compile`
- D: `dbt deps`

Answer: B. `dbt source freshness` validates recency.

1. How should you manage third-party package versions safely?

- A: Use moving branches
- B: Pin versions and read changelogs
- C: Always latest
- D: Avoid packages

Answer: B. Pin versions and upgrade intentionally.

1. Where do you declare packages for a project?

- A: dbt_project.yml
- B: packages.yml
- C: profiles.yml
- D: schema.yml

Answer: B. `packages.yml` declares dependencies.

1. Which field powers freshness checks?

- A: `owner`
- B: `loaded_at_field`
- C: `grants`
- D: `access`

Answer: B. `loaded_at_field` is used to measure recency.

[Back to README](./README.md) • Prev: [06 — Documentation](./06-documentation.md) • Next: [08 — Leveraging State](./08-leveraging-state.md)
