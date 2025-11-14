# Security and Compliance

## Concept Definition

Security and compliance are fundamental aspects of the Databricks platform. Databricks provides a comprehensive set of features to help you secure your data and meet your compliance requirements. This includes features for authentication, access control, data encryption, and auditing. [1]

## Key Security and Compliance Features

- **Unity Catalog**: A unified governance solution for data and AI assets in the Databricks Lakehouse. It provides centralized access control, auditing, and data discovery. [2]
- **Access Control Lists (ACLs)**: Used to configure permissions for workspace objects like notebooks, jobs, and clusters.
- **Data Encryption**: Databricks encrypts data at rest and in transit. You can also use your own keys for encryption.
- **Secret Management**: Databricks secrets allow you to securely store and manage your credentials for accessing external data sources.
- **Compliance Certifications**: Databricks maintains a number of compliance certifications, such as SOC 2, HIPAA, and PCI DSS, to help you meet your regulatory requirements. [3]

## Examples

### Granting Access to a Table in Unity Catalog

```sql
GRANT SELECT ON TABLE my_catalog.my_schema.my_table TO `user@example.com`;
```

### Creating a Secret

```bash
databricks secrets create-scope --scope my_scope
databricks secrets put --scope my_scope --key my_key --string-value my_secret_value
```

## Best Practices

- **Use Unity Catalog for Governance**: Use Unity Catalog to centralize the governance of your data and AI assets.
- **Follow the Principle of Least Privilege**: Grant users and service principals only the permissions they need to perform their jobs.
- **Use Service Principals for Automation**: Use service principals to run your automated jobs and pipelines.
- **Regularly Audit Your Environment**: Use audit logs and system tables to regularly audit your Databricks environment for security and compliance.

## Gotchas

- **Unity Catalog is Not a Silver Bullet**: While Unity Catalog provides a powerful set of governance features, it is not a replacement for a comprehensive security and compliance strategy.
- **Permissions Can Be Complex**: Managing permissions in a large organization can be complex. Use groups to simplify permission management.
- **Compliance is a Shared Responsibility**: Databricks provides a secure and compliant platform, but you are ultimately responsible for ensuring that your use of the platform meets your regulatory requirements.

## Mock Questions

1. **Which Databricks feature provides a unified governance solution for data and AI assets?**

    a. DBFS  
    b. Unity Catalog  
    c. The Spark UI  
    d. Databricks SQL

2. **What is the recommended way to store credentials for accessing external data sources?**

    a. Hardcode them in your notebooks.  
    b. Store them in a plain text file.  
    c. Use Databricks secrets.  
    d. Email them to your colleagues.

3. **What is the principle of least privilege?**

    a. Granting all users and service principals the same level of access.  
    b. Granting users and service principals only the permissions they need to perform their jobs.  
    c. Granting no permissions to any users or service principals.  
    d. Granting all permissions to all users and service principals.

## Answers

1. b
2. c
3. b

## References

[1] [Security and compliance | Databricks on AWS](https://docs.databricks.com/aws/en/security/)
[2] [What is Unity Catalog?](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/)
[3] [Databricks Trust & Compliance: Ensuring Security & Privacy](https://www.databricks.com/trust/compliance)
