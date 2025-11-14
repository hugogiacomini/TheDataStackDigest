# Data Governance

## Concept Definition

Data governance is the overall management of the availability, usability, integrity, and security of the data in an enterprise. In Databricks, data governance is primarily achieved through **Unity Catalog**, which provides a unified governance solution for data and AI assets in the Databricks Lakehouse. [1]

## Key Data Governance Features in Unity Catalog

- **Centralized Metadata Management**: A single place to store and manage metadata for all your data assets.
- **Centralized Access Control**: A single place to define and manage access control policies for all your data assets.
- **Data Lineage**: The ability to track the lineage of your data, from source to destination.
- **Data Discovery**: The ability to search and discover data assets across your entire organization.
- **Auditing**: The ability to audit access to your data and AI assets.

## Examples

### Viewing Data Lineage

You can view the lineage of a table in the Catalog Explorer. The lineage graph shows you the upstream and downstream dependencies of the table.

### Searching for Data

You can use the search bar in the Databricks UI to search for data assets across your entire workspace.

## Best Practices

- **Embrace Unity Catalog**: Unity Catalog is the foundation of data governance in Databricks. Embrace it and use it to its full potential.
- **Define a Clear Governance Model**: Define a clear data governance model that outlines the roles and responsibilities for data governance in your organization.
- **Use Tags to Organize Your Data**: Use tags to organize your data assets and make them easier to discover.
- **Regularly Review Your Governance Policies**: Regularly review your data governance policies to ensure they are still relevant and effective.

## Gotchas

- **Unity Catalog is a Journey, Not a Destination**: Implementing data governance with Unity Catalog is a journey, not a one-time project. It requires ongoing effort and commitment.
- **Data Quality is Key**: Data governance is not just about access control and lineage. It is also about ensuring the quality of your data.
- **User Adoption is Crucial**: The success of your data governance program depends on user adoption. Be sure to provide training and support to your users.

## Mock Questions

1. **Which Databricks feature is the foundation of data governance in the Databricks Lakehouse?**

    a. DBFS  
    b. Unity Catalog  
    c. The Spark UI  
    d. Databricks SQL

2. **What is data lineage?**

    a. The process of deleting old data.  
    b. The ability to track the lineage of your data, from source to destination.  
    c. The process of granting access to data.  
    d. The process of creating new tables.

3. **What is a key benefit of using tags in Unity Catalog?**

    a. To encrypt your data.  
    b. To organize your data assets and make them easier to discover.  
    c. To delete your data.  
    d. To grant access to your data.

## Answers

1. b
2. b
3. b

## References

[1] [Data governance with Databricks](https://docs.databricks.com/aws/en/data-governance/)
