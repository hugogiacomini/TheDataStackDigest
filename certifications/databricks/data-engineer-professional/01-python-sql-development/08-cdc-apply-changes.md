# CDC with APPLY CHANGES API

## Concept Definition

Change Data Capture (CDC) is a data integration pattern used to capture changes made to data in a source system and apply those changes to a target system. In Databricks, the `APPLY CHANGES` API in Lakeflow Declarative Pipelines simplifies CDC by providing a declarative way to handle inserts, updates, and deletes from a source table to a target Delta table. [1]

**Note:** While the `APPLY CHANGES` API is still available, Databricks now recommends using the `AUTO CDC` APIs, which have a similar syntax but offer additional benefits. [2]

## Key Features

- **Declarative Syntax**: Simplifies CDC logic with a high-level, declarative API.
- **Handles Out-of-Sequence Data**: Automatically handles records that arrive out of order.
- **SCD Type 1 and Type 2**: Supports both Slowly Changing Dimension (SCD) Type 1 (overwrite) and Type 2 (history tracking) updates.
- **Simplified MERGE Operations**: Provides a more straightforward way to perform `MERGE` operations for CDC.

## Python/PySpark Examples

### Applying Changes in Python

```python
import dlt

@dlt.table
def target_table():
  return spark.read.format("delta").load("/path/to/target/table")

dlt.apply_changes(
  target = "target_table",
  source = "source_table",
  keys = ["id"],
  sequence_by = "timestamp",
  stored_as_scd_type = 1
)
```

## SQL Examples

### Applying Changes in SQL

```sql
APPLY CHANGES INTO LIVE.target_table
FROM stream(LIVE.source_table)
KEYS (id)
SEQUENCE BY timestamp
STORED AS SCD TYPE 1;
```

## Best Practices

- **Use `sequence_by`**: Always use the `sequence_by` clause to ensure that changes are applied in the correct order, especially when dealing with out-of-order data.
- **Choose the Right SCD Type**: Select the appropriate SCD type (1 or 2) based on your business requirements for handling historical data.
- **Use with Streaming Data**: The `APPLY CHANGES` API is designed to work with streaming data. Use `stream(LIVE.source_table)` in SQL to read the source table as a stream.

## Gotchas

- **`AUTO CDC` is Recommended**: While `APPLY CHANGES` is still functional, `AUTO CDC` is the newer, recommended approach. Be aware of this for future projects.
- **Primary Keys**: The `keys` clause specifies the primary key(s) used to match records between the source and target tables.
- **Streaming Source**: The `APPLY CHANGES` API expects a streaming source. If your source is a batch source, you may need to use a different approach, such as a `MERGE` statement.

## Mock Questions

1.  **What is the purpose of the `sequence_by` clause in the `APPLY CHANGES` API?**
    a.  To specify the primary key of the table.
    b.  To ensure that changes are applied in the correct order.
    c.  To define the SCD type.
    d.  To filter the source data.

2.  **Which of the following is a key benefit of the `APPLY CHANGES` API?**
    a.  It can only be used with batch data.
    b.  It simplifies CDC logic with a declarative syntax.
    c.  It requires you to manually handle out-of-sequence data.
    d.  It does not support SCD Type 2.

3.  **What is the recommended way to handle CDC in Databricks as of the latest updates?**
    a.  Using the `MERGE` statement.
    b.  Using the `APPLY CHANGES` API.
    c.  Using the `AUTO CDC` APIs.
    d.  Writing a custom Python script.

**Answers:**
1.  b
2.  b
3.  c

## References

[1] [Change Data Capture With Delta Live Tables](https://www.databricks.com/blog/2022/04/25/simplifying-change-data-capture-with-databricks-delta-live-tables.html)
[2] [The AUTO CDC APIs: Simplify change data capture with Lakeflow Declarative Pipelines](https://docs.databricks.com/aws/en/ldp/cdc)
