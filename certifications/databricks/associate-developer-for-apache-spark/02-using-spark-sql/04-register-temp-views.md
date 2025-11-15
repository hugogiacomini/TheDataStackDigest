# Register DataFrames as Temporary Views

## Overview

Create session or global temp views allowing SQL queries on in-memory DataFrames.

## Temp View Types

- Session-scoped: `createOrReplaceTempView("view_name")`.  
- Global-scoped: `createOrReplaceGlobalTempView("view_name")` → access via `global_temp.view_name`.

## Example Workflow

```python
# Load DataFrame
events = spark.read.json("/mnt/data/events/")

# Register temp view
events.createOrReplaceTempView("events_view")

# Query with SQL
high_value = spark.sql("""
    SELECT user_id, COUNT(*) AS event_count
    FROM events_view
    WHERE event_type = 'purchase'
    GROUP BY user_id
    HAVING event_count > 5
""")

high_value.show()
```

## Global Temp View

```python
events.createOrReplaceGlobalTempView("events_global")
spark.sql("SELECT * FROM global_temp.events_global LIMIT 10").show()
```

## Best Practices

- Use temp views for intermediate SQL logic within notebooks.  
- Prefer persistent tables (managed/external) for shared access across sessions.  
- Name views descriptively to avoid namespace collisions.

## Sample Questions

1. Lifespan of a temp view?  
2. How access global temp view?  
3. When prefer temp view over persisted table?  
4. Can temp views survive Spark session restart?  
5. How replace existing temp view?

## Answers

1. Lasts until session ends.  
2. Via `global_temp.<view_name>`.  
3. For ephemeral intermediate results within single session/notebook.  
4. No; they are session-scoped.  
5. Use `createOrReplaceTempView` (overwrites if exists).

## References

- [Temp Views](https://spark.apache.org/docs/latest/sql-getting-started.html#creating-datasets)

---

Previous: [Persistent Tables & Sorting](./03-persistent-tables-sorting-partitioning.md)  
Next: [DataFrame Column & Row Manipulation](../03-developing-dataframe-dataset-api-applications/01-column-row-manipulation.md)
