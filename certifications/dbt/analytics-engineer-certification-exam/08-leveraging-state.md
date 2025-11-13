# 08 — Leveraging the dbt State

## Key Concepts and Definitions

- State: Use artifacts (manifest/run_results) from a prior run to select only changed/impacted models.
- Selectors: `state:modified`, `state:modified+`, `result:error`, etc.
- Defer: Resolve `ref()` to upstream environment artifacts for faster CI (`--defer --state <path>`).
- Retry: Re-run failed models automatically (`dbt retry`) to handle transient issues.

## Practical Examples with Code

### 1. Slim CI with State and Defer

```bash
# Assume artifacts/prod contains manifest from last prod deploy
 dbt run --select state:modified+ --defer --state artifacts/prod
 dbt test --select state:modified+ --defer --state artifacts/prod --fail-fast
```

- `state:modified+` picks changed models and their children.
- `--defer` uses prod artifacts for unresolved refs, avoiding full rebuilds.

### 2. Selectors for Targeted Runs

```bash
# Only models changed since previous manifest
 dbt run --select state:modified --state target

# Only models that errored in the previous run
 dbt run --select result:error --state target
```

### 3. dbt retry for Transient Failures

```bash
# Re-run only models that failed previously
 dbt retry --target prod
```

### 4. Combining State with Tags and Paths

```bash
# Only modified models in marts
 dbt run -s state:modified+ @marts --state artifacts/prod

# Only modified critical models
 dbt run -s state:modified+ tag:critical --state artifacts/prod
```

## Best Practices and Anti-Patterns

- Store and version artifacts from prod in CI to enable deferral and slim runs.
- Combine selectors logically to minimize compute and blast radius.
- Use `retry` judiciously; investigate systemic failures rather than looping indefinitely.
- Avoid using state artifacts that are out-of-sync with warehouse data; refresh regularly.

## Real-World Scenarios and Solutions

- Fast PR validation: Use `state:modified+` with defer to test only impacted surface.
- Partial prod incident: `result:error` selects failed scope; rerun after transient fix.
- Cost pressure: State-based selection reduces compute by 70–90% vs full DAG.

## Sample Exam Questions

1. What does `--defer` do in combination with `--state`?

- A: Rebuilds all models
- B: Resolves refs to upstream environment artifacts
- C: Disables tests
- D: Runs seeds only

Answer: B. Deferral resolves refs to artifact-defined relations.

1. How do you run only changed models and their downstream dependents?

- A: `state:modified+`
- B: `+state:modified`
- C: `tag:modified+`
- D: `path:modified+`

Answer: A. The `+` suffix includes children (downstream).

1. Which selector re-runs models that failed previously?

- A: `state:error`
- B: `result:error`
- C: `state:failed`
- D: `result:failed`

Answer: B. `result:error` selects models failed in prior results.

1. What’s a key benefit of slim CI?

- A: Always full-refresh models
- B: Faster runs by executing only impacted models
- C: Disabling tests
- D: Avoiding version control

Answer: B. Slim CI reduces runtime and cost.

1. When can `dbt retry` be useful?

- A: When failures are transient
- B: To rerun entire DAGs
- C: To change contracts
- D: To deploy docs

Answer: A. Retry focuses on previously failed nodes.

[Back to README](./README.md) • Prev: [07 — External Dependencies](./07-external-dependencies.md)
