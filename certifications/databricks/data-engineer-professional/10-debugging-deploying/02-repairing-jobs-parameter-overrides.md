# Repairing Jobs and Parameter Overrides

## Overview

When a production job fails, you often need to re-run only failed tasks, tweak parameters, or override inputs without reprocessing everything. Databricks Workflows (Jobs) support job repair, task retries, and parameter overrides via UI, CLI, and REST APIs.

## Concepts

- Job repair: re-run only failed tasks (and their downstreams) of a prior run.
- Parameters: job-level and task-level parameters (widgets, base_parameters) that can be overridden at submission time.
- Idempotency: design tasks to safely re-run without duplicating side effects.

## UI and CLI Workflows

### UI (Job repair)

1. Workflows → Jobs → Select failed run.  
2. Click Repair.  
3. Optionally override parameters; re-run only failed graph.

### CLI (Jobs API via Databricks CLI v0)

```bash
# Trigger a run with parameter overrides
databricks jobs run-now \
  --job-id 1234 \
  --notebook-params '{"run_date":"2025-11-01","full_refresh":"false"}'

# Repair a failed run (re-run only failed tasks)
databricks jobs repair-run \
  --run-id 567890 \
  --rerun-all-failed-tasks true
```

### REST API (Jobs 2.1)

```bash
curl -s -X POST "$DATABRICKS_HOST/api/2.1/jobs/run-now" \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
        "job_id": 1234,
        "notebook_params": {"run_date": "2025-11-01", "full_refresh": "false"}
      }'

curl -s -X POST "$DATABRICKS_HOST/api/2.1/jobs/repair-run" \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
        "run_id": 567890,
        "rerun_all_failed_tasks": true,
        "latest_repair_id": 1
      }'
```

## Patterns and Anti-Patterns

- Prefer input-driven idempotency over manual cleanups (partition overwrite by date).
- Use parameterized paths and dates; default from cluster env or job params.
- Keep retries on transient failures; avoid indefinite retries for logic errors.

```python
# Example: task-level parameter handling in notebook
dbutils.widgets.text("run_date", "")
dbutils.widgets.dropdown("full_refresh", "false", ["true","false"]) 

run_date = dbutils.widgets.get("run_date") or spark.sql("select date_sub(current_date(),1)").first()[0].isoformat()
full_refresh = dbutils.widgets.get("full_refresh") == "true"

if full_refresh:
    mode = "overwrite"
else:
    mode = "append"

df = load_source(run_date)
df.write.mode(mode).format("delta").saveAsTable("silver.fact_orders")
```

## Troubleshooting

- “Task already running” during repair → ensure previous attempt finished/cancelled.
- Partial writes → use Delta transactional writes (`saveAsTable`) and partition overwrite.
- Dependency drift → pin notebook and library versions; use Bundles for deployment.

## Mock Questions

1. **Job repair re-runs which subset of tasks by default?**  
    a. Entire DAG  
    b. Only upstream dependencies  
    c. Only failed tasks and dependents  
    d. Only the root task

2. **Which is the safest way to avoid duplicates during a repair?**  
    a. Use `append` blindly  
    b. Parameterize by date partition and overwrite that partition  
    c. Delete the target table  
    d. Increase retries to 10

3. **Which API can re-run only failed tasks of a prior run?**  
    a. `jobs/run-now`  
    b. `jobs/repair-run`  
    c. `runs/submit`  
    d. `pipelines/deploy`

## Answers

1. c
2. b
3. b

## References

- Jobs API 2.1 (run-now, repair-run)
- Workflows UI – Job repairs and parameters
- Delta Lake partition overwrite and transactional writes
