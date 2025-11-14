# Data Sharing and Federation

## Concept Definition

Databricks provides two primary mechanisms for sharing and accessing data across different platforms and systems: **Delta Sharing** and **Lakehouse Federation**.

- **Delta Sharing** is an open protocol for securely sharing live data from your Databricks lakehouse with other organizations, regardless of the computing platform they use. It allows you to share data without copying or moving it, ensuring that consumers always have access to the latest version. [1]

- **Lakehouse Federation** is a feature that allows you to query data in external data sources (such as PostgreSQL, MySQL, and other Databricks workspaces) as if it were stored in your own Databricks workspace. It enables you to run federated queries that span multiple data sources. [2]

## Key Differences

| Feature | Delta Sharing | Lakehouse Federation |
|---|---|---|
| **Purpose** | Sharing data with external consumers | Querying data in external systems |
| **Direction** | Outbound (sharing data from your workspace) | Inbound (querying data from external systems) |
| **Data Movement** | No data movement (live access) | No data movement (queries are pushed down) |
| **Protocol** | Open protocol | Proprietary |

## Examples

### Sharing a Table with Delta Sharing

```sql
-- Create a share
CREATE SHARE my_share;

-- Add a table to the share
ALTER SHARE my_share ADD TABLE my_table;

-- Grant access to a recipient
GRANT SELECT ON SHARE my_share TO RECIPIENT my_recipient;
```

### Creating a Connection for Lakehouse Federation

```sql
-- Create a connection to a PostgreSQL database
CREATE CONNECTION my_postgres_connection
TYPE postgresql
OPTIONS (
  host 'my-postgres-host',
  port '5432',
  user 'my_user',
  password 'my_password'
);
```

## Best Practices

- **Use Delta Sharing for External Collaboration**: When you need to share data with external partners or customers, Delta Sharing is the recommended approach.
- **Use Lakehouse Federation for Data Virtualization**: When you need to query data in external systems without moving it, Lakehouse Federation is the ideal solution.
- **Secure Your Connections**: When using Lakehouse Federation, use Databricks secrets to store your credentials for external data sources.

## Gotchas

- **Permissions**: For both Delta Sharing and Lakehouse Federation, you need to ensure that you have the correct permissions to share or access the data.
- **Performance**: While Lakehouse Federation pushes down queries to the external data source, there can still be performance overhead. Be mindful of the performance of your federated queries.
- **Data Types**: Not all data types may be supported across all data sources in Lakehouse Federation.

## Mock Questions

1. **Which Databricks feature would you use to share a live dataset with an external partner who is not on Databricks?**

    a. Lakehouse Federation  
    b. Delta Sharing  
    c. DBFS  
    d. Unity Catalog

2. **What is the primary purpose of Lakehouse Federation?**

    a. To share data with external consumers.  
    b. To query data in external data sources without moving it.  
    c. To create copies of data in your Databricks workspace.  
    d. To manage permissions on your data.

3. **What is a key benefit of Delta Sharing?**

    a. It requires you to copy data to a shared location.  
    b. It is a proprietary protocol.  
    c. It allows you to share live data without copying or moving it.  
    d. It only works with other Databricks workspaces.

## Answers

1. b
2. b
3. c

## References

[1] [What is Delta Sharing? | Databricks on AWS](https://docs.databricks.com/aws/en/delta-sharing/)
[2] [What is Lakehouse Federation? | Databricks on AWS](https://docs.databricks.com/aws/en/query-federation/)
