# Managing Libraries in Databricks

## Concept Definition

In Databricks, libraries allow you to make third-party or custom code available to your notebooks and jobs. You can install libraries at different scopes, primarily **compute-scoped** and **notebook-scoped**. [1]

- **Compute-scoped libraries** are installed on a compute resource (cluster) and are available to all notebooks and jobs running on that compute. They are the recommended approach for setting up a consistent environment for multiple users and jobs.

- **Notebook-scoped libraries** are installed within a specific notebook session and are isolated to that notebook. They do not affect other notebooks on the same compute. These libraries do not persist across sessions and must be re-installed each time.

## Key Features

Libraries can be installed from various sources:

| Source | Description |
|---|---|
| **Package Repository** | Public repositories like PyPI, Maven, or CRAN. |
| **Workspace Files** | Library files (e.g., `.whl`, `.jar`) uploaded to your Databricks workspace. |
| **Unity Catalog Volumes** | Library files stored in Unity Catalog volumes, providing centralized governance. |
| **Cloud Object Storage** | Library files stored in cloud object storage like S3, ADLS Gen2, or GCS. |
| **Local Machine** | Files from your local machine uploaded directly to the cluster. |

## Python/PySpark Examples

### Installing Notebook-Scoped Libraries

The recommended way to install notebook-scoped Python libraries is using the `%pip` magic command.

```python
%pip install pandas
```

To install a specific version:

```python
%pip install pandas==1.5.3
```

To install from a `requirements.txt` file:

```python
%pip install -r /path/to/requirements.txt
```

### Installing Compute-Scoped Libraries

Compute-scoped libraries are typically installed through the Databricks UI:

1.  Navigate to the **Compute** page and select your cluster.
2.  Click the **Libraries** tab.
3.  Click **Install New**.
4.  Select the library source (e.g., PyPI) and follow the instructions.

## Best Practices

- **Use `requirements.txt`**: For reproducible environments, manage your Python dependencies using a `requirements.txt` file. This is supported for both notebook-scoped and compute-scoped libraries (DBR 15.0+).
- **Choose the Right Scope**: Use compute-scoped libraries for common dependencies across multiple notebooks and jobs. Use notebook-scoped libraries for ad-hoc or specific needs of a single notebook.
- **Use Workspace Files or Volumes**: For custom libraries or when you need more control, store your library files in workspace files or Unity Catalog volumes. This is more secure than using DBFS.
- **Service Principals for Jobs**: When scheduling jobs, run them with a service principal to avoid issues with user-based authentication.

## Gotchas

- **DBFS Deprecation**: Storing library files in the DBFS root is deprecated and disabled by default in DBR 15.1 and above due to security concerns. [1]
- **Library Conflicts**: Be mindful of library versions. Conflicts can arise if different notebooks on the same cluster require different versions of the same library. Notebook-scoped libraries can help mitigate this.
- **JARs at Notebook Level**: You cannot install JAR files at the notebook level. They must be installed as compute-scoped libraries.
- **Restarting Python Process**: After installing or uninstalling libraries with `%pip`, you may need to restart the Python process to use the new libraries. Databricks provides a `dbutils.library.restartPython()` command for this.

## Mock Questions

1.  **What is the recommended way to install a Python library for a single notebook session?**
    a.  Install it as a compute-scoped library.
    b.  Use the `%pip` magic command.
    c.  Store the library in the DBFS root.
    d.  Email the library to your administrator.

2.  **Which of the following is a key benefit of using notebook-scoped libraries?**
    a.  They are automatically available to all notebooks on the cluster.
    b.  They persist across all notebook sessions.
    c.  They provide an isolated environment for a specific notebook, preventing conflicts.
    d.  They can be used to install JAR files.

3.  **Why is storing libraries in the DBFS root deprecated?**
    a.  It is slower than other methods.
    b.  It has a smaller storage capacity.
    c.  It poses a security risk as any user can modify the files.
    d.  It only supports Python libraries.

**Answers:**
1.  b
2.  c
3.  c

## References

[1] [Install libraries | Databricks on AWS](https://docs.databricks.com/aws/en/libraries/)
