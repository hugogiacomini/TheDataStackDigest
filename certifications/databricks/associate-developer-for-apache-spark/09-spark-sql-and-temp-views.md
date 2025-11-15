# Spark SQL and Temp Views

## Overview

Use temporary and global temporary views to blend SQL with DataFrame APIs and facilitate modular logic.

## Prerequisites

- DataFrame creation and basic SQL syntax.

## Concepts

- Temp view: Session-scoped; `createOrReplaceTempView`.
- Global temp view: Scoped to `global_temp` database; accessible across sessions in same app.
- SQL interoperability: Switch between DataFrame DSL and SQL for expressiveness.
- Catalog vs temp views: Persisted tables vs ephemeral logical views.

## Hands-on Walkthrough

### Create and Query Temp View

```python
df = spark.read.parquet("/mnt/data/events/")
df.createOrReplaceTempView("events")
sql_df = spark.sql("SELECT event_type, COUNT(*) AS c FROM events GROUP BY event_type")
```

### Global Temp View

```python
df.createOrReplaceGlobalTempView("events_global")
spark.sql("SELECT * FROM global_temp.events_global LIMIT 5").show()
```

### Mixed API Usage

```python
intermediate = spark.sql("SELECT * FROM events WHERE event_type = 'click'")\
                   .groupBy("event_type").count()
```

## Production Considerations

- Avoid overuse of temp views; DataFrame chaining often clearer.
- Global temp views persist until Spark application ends; manage lifecycle.
- For sharing across notebooks/jobs → prefer persisted tables or Delta.

## Troubleshooting

- View not found: Session scope ended or name mismatch.
- Stale view schema: Recreate after upstream schema change.
- Performance: Complex SQL may hide inefficient plan; use `EXPLAIN`.

## Sample Questions

1. Difference between temp view and global temp view?  
2. When prefer DataFrame API over SQL?  
3. Where are global temp views stored?  
4. How to refresh a view after schema change?  
5. What command inspects SQL physical plan?

## Answers

1. Temp view is session-scoped; global temp view lives in `global_temp` database across sessions.  
2. For programmatic composition/operator chaining clarity.  
3. In the `global_temp` system database.  
4. Recreate view with `createOrReplaceTempView`.  
5. `EXPLAIN` or `df.explain(True)`.

## References

- [Spark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [Temp Views](https://spark.apache.org/docs/latest/sql-ref-syntax-ddl-create-view.html)

---

Previous: [User-Defined Functions and Built-ins](./08-user-defined-functions-and-builtins.md)  
Next: [Structured Streaming Basics](./10-structured-streaming-basics.md)
