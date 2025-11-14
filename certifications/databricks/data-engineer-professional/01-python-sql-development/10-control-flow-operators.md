
# Control Flow Operators

## Concept Definition

In Databricks, control flow operators allow you to define the execution logic of your workflows, enabling you to create complex and dynamic data pipelines. These operators are primarily used within **Lakeflow Jobs** (formerly Databricks Workflows) to orchestrate the execution of tasks. [1]

The main control flow capabilities are:

- **Task Dependencies**: Defining the order in which tasks run, either sequentially or in parallel.
- **Conditional Execution (`If/else condition`)**: Running different tasks based on the outcome of a boolean condition.
- **Looping (`For each` loop)**: Running a task multiple times for each item in a list.

## Key Features

- **Build Complex DAGs**: Create sophisticated Directed Acyclic Graphs (DAGs) that represent your data processing logic.
- **Dynamic Workflows**: Make your workflows more dynamic and responsive to different conditions and inputs.
- **Increased Reusability**: Use loops to reuse the same task for different inputs, reducing code duplication.
- **Improved Error Handling**: Implement conditional logic to handle task failures and other exceptional situations.

## Examples

Control flow is typically defined in the Jobs UI or through the Jobs API using JSON. Here are conceptual examples of how these operators are used.

### Conditional Execution (`If/else condition`)

This allows you to branch your workflow based on a condition. For example, you could run a data quality check and then, based on the result, either proceed with the pipeline or run a cleanup task.

```json
{
  "name": "Conditional Workflow",
  "tasks": [
    {
      "task_key": "data_quality_check",
      "notebook_task": { ... }
    },
    {
      "task_key": "process_data",
      "depends_on": [{ "task_key": "data_quality_check" }],
      "run_if": "ALL_SUCCESS"
    },
    {
      "task_key": "cleanup_task",
      "depends_on": [{ "task_key": "data_quality_check" }],
      "run_if": "ANY_FAILED"
    }
  ]
}
```

### Looping (`For each` loop)

This allows you to run a task for each item in a list. For example, you could have a list of tables to process and run the same notebook for each table.

```json
{
  "name": "Looping Workflow",
  "tasks": [
    {
      "task_key": "process_tables",
      "forEach": {
        "items": ["table1", "table2", "table3"],
        "task": {
          "notebook_task": {
            "notebook_path": "/Users/user@example.com/ProcessTable",
            "base_parameters": {
              "table_name": "{{item}}"
            }
          }
        }
      }
    }
  ]
}
```

## Best Practices

- **Keep Logic Clear**: Design your workflows to be as clear and understandable as possible. Use meaningful task names and comments.
- **Parameterize Notebooks**: When using loops, parameterize your notebooks to accept the loop variable as an input.
- **Handle Failures Gracefully**: Use conditional execution to handle task failures and prevent your entire workflow from failing.
- **Monitor Execution**: Regularly monitor the execution of your workflows to identify any bottlenecks or issues.

## Gotchas

- **Complex Nesting**: Avoid overly complex nesting of conditional logic and loops, as it can make your workflow difficult to understand and debug.
- **Infinite Loops**: Be careful when defining the items for a `For each` loop to avoid creating an infinite loop.
- **Task Concurrency**: Be mindful of the number of parallel tasks you are running, as it can impact the performance of your cluster.

## Mock Questions

1. **Which control flow operator would you use to run a task multiple times with different inputs?**

    a.  `If/else condition`  
    b.  `For each` loop  
    c.  Task Dependencies  
    d.  `Run if`  

2. **What is the primary purpose of using control flow operators in Databricks Jobs?**

    a.  To write Python code.  
    b.  To orchestrate the execution of tasks in a workflow.  
    c.  To create Delta tables.  
    d.  To manage cluster permissions.  

3. **How can you handle task failures in a Databricks Job workflow?**

    a.  By using a `For each` loop.  
    b.  By using conditional execution (`If/else condition`) to run a cleanup task if a previous task fails.  
    c.  By increasing the cluster size.  
    d.  By deleting the failed task.  

## Answers

1. b
2. b
3. b

## References

[1] [Control the flow of tasks within Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/control-flow)
