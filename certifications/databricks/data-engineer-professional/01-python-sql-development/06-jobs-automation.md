# Jobs Automation

## Concept Definition

Databricks Jobs is a service for running non-interactive workloads in a Databricks workspace. A job can consist of a single task or a complex workflow with multiple tasks and dependencies. Jobs can be triggered manually, on a schedule, or by external events, making them a powerful tool for automating data engineering pipelines. [1]

## Key Features

- **Multiple Task Types**: Jobs can run notebooks, Python scripts, JARs, and more.
- **Task Dependencies**: You can define dependencies between tasks to create complex workflows.
- **Scheduling**: Jobs can be scheduled to run at specific times or intervals.
- **Alerts and Notifications**: You can configure alerts to be notified of job status changes.
- **Job Clusters**: Jobs can run on dedicated job clusters that are created for the duration of the job and terminated upon completion, which can be more cost-effective than using all-purpose clusters.

## Python/PySpark Examples

### Creating a Job using the Databricks CLI

First, define your job in a JSON file (e.g., `my-job.json`):

```json
{
  "name": "My CLI Job",
  "tasks": [
    {
      "task_key": "my_notebook_task",
      "notebook_task": {
        "notebook_path": "/Users/user@example.com/MyNotebook"
      },
      "new_cluster": {
        "spark_version": "14.3.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    }
  ]
}
```

Then, create the job using the Databricks CLI:

```bash
databricks jobs create --json-file my-job.json
```

### Creating a Job using the REST API

You can use the Databricks Jobs API to create and manage jobs programmatically. Here's an example using `curl`:

```bash
curl -X POST -H "Authorization: Bearer <your-token>" \
  https://<your-databricks-instance>/api/2.1/jobs/create \
  -d @my-job.json
```

## Best Practices

- **Use Job Clusters**: For scheduled or automated jobs, use job clusters to reduce costs. Job clusters are terminated when the job is complete.
- **Use Service Principals**: Run your jobs as a service principal to avoid issues with user-based authentication and permissions.
- **Monitor Job Runs**: Use the Jobs UI, API, or system tables to monitor the status and performance of your jobs.
- **Use Git for Source Control**: Store your job definitions and code in a Git repository for version control and collaboration.

## Gotchas

- **API Versioning**: The Databricks Jobs API has different versions (e.g., 2.0, 2.1). Be sure to use the correct version for the features you need.
- **Permissions**: Ensure that the user or service principal running the job has the necessary permissions to access the required resources (e.g., notebooks, data).
- **Cluster Configuration**: Incorrectly configured clusters (e.g., insufficient memory, incorrect Spark version) can cause job failures.
- **Timeouts**: Jobs can time out if they run for longer than the configured timeout period.

## Mock Questions

1.  **What is a key advantage of using job clusters for scheduled jobs?**
    a.  They are always running and ready to execute jobs.
    b.  They are more expensive than all-purpose clusters.
    c.  They are created for the duration of the job and terminated upon completion, which can be more cost-effective.
    d.  They can only run notebook tasks.

2.  **Which of the following is a recommended best practice for automating jobs in Databricks?**
    a.  Running jobs as your personal user account.
    b.  Storing job definitions on your local machine.
    c.  Using a service principal to run jobs.
    d.  Using all-purpose clusters for all jobs.

3.  **What is a common way to define a Databricks job for automation?**
    a.  In a Word document.
    b.  In a JSON file.
    c.  In an email.
    d.  In a text message.

**Answers:**
1.  c
2.  c
3.  b

## References

[1] [Lakeflow Jobs | Databricks on AWS](https://docs.databricks.com/aws/en/jobs/)
