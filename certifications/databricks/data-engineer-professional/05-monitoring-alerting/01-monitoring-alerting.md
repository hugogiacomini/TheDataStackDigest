# Monitoring and Alerting

## Concept Definition

Monitoring and alerting are critical components of a robust data engineering ecosystem. In Databricks, this involves tracking the health and performance of your jobs, clusters, and data pipelines, and setting up alerts to be notified of important events or issues. [1]

## Key Monitoring and Alerting Tools

- **Jobs UI**: The Jobs UI provides a visual interface for monitoring the status and history of your job runs.
- **Spark UI**: The Spark UI provides detailed information about the performance of your Spark jobs.
- **System Tables**: System tables are a Databricks-hosted analytical store of your account’s operational data found in the `system` catalog. They can be used to monitor a variety of aspects of your account, including job performance, cluster status, and user activity. [2]
- **Databricks SQL Alerts**: Databricks SQL alerts allow you to periodically run a query, evaluate a condition, and send a notification if the condition is met. [3]
- **Query History**: The query history in Databricks SQL allows you to view the history of all queries run in your workspace.

## Examples

### Querying Job Status from System Tables

```sql
SELECT
  job_id,
  job_name,
  run_id,
  status,
  start_time,
  end_time
FROM system.jobs.runs
WHERE job_id = 12345
ORDER BY start_time DESC;
```

### Creating a Databricks SQL Alert

1. Create a query that returns a value you want to monitor.
2. Click the "Create Alert" button.
3. Configure the alert to trigger when the value meets a certain condition (e.g., is greater than a threshold).
4. Configure the alert to send a notification to an email address or other destination.

## Best Practices

- **Use System Tables for Historical Analysis**: System tables are a powerful tool for analyzing the historical performance of your jobs and clusters.
- **Use Databricks SQL Alerts for Proactive Monitoring**: Use Databricks SQL alerts to be proactively notified of issues before they become critical.
- **Monitor Costs**: Use system tables and other tools to monitor the cost of your Databricks usage.

## Gotchas

- **System Tables are in Preview**: System tables are currently in public preview. Be aware that the schema and functionality may change.
- **Alert Frequency**: Be mindful of the frequency of your alerts. Alerts that are too frequent can be noisy and may be ignored.
- **Permissions**: You need the appropriate permissions to view system tables and create alerts.

## Mock Questions

1. **Which Databricks feature would you use to get a historical view of your job runs?**

    a. The Jobs UI  
    b. The Spark UI  
    c. System tables  
    d. Databricks SQL alerts

2. **What is the primary purpose of Databricks SQL alerts?**

    a. To run Spark jobs.  
    b. To periodically run a query, evaluate a condition, and send a notification if the condition is met.  
    c. To view the history of all queries run in your workspace.  
    d. To create Delta tables.

3. **What is a potential issue with system tables?**

    a. They are not available in all regions.  
    b. They are difficult to query.  
    c. They are in public preview and the schema may change.  
    d. They do not contain historical data.

## Answers

1. c
2. b
3. c

## References

[1] [Monitoring and observability for Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/monitor)
[2] [Monitor job costs & performance with system tables](https://docs.databricks.com/aws/en/admin/system-tables/jobs-cost)
[3] [Databricks SQL alerts](https://docs.databricks.com/aws/en/sql/user/alerts/)
