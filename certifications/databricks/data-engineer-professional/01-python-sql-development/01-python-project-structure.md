# Databricks Asset Bundles (DABs) Project Structure

## Concept Definition

Databricks Asset Bundles (DABs) provide an Infrastructure-as-Code (IaC) approach to managing Databricks projects. They facilitate the adoption of software engineering best practices, including source control, code review, testing, and continuous integration and delivery (CI/CD). A bundle is an end-to-end definition of a project, including its structure, tests, and deployment configurations, making it easier to collaborate on projects during active development. [1]

## Key Features

- **Modular Development**: Bundles help organize and manage various source files efficiently, ensuring smooth collaboration and streamlined processes.
- **Deployment Automation**: Bundles can be deployed programmatically, enabling automated and repeatable deployments across different environments.
- **CI/CD Integration**: Bundles are designed to be integrated into CI/CD pipelines, allowing for automated testing and deployment of Databricks projects.
- **Resource Management as Code**: Bundles allow you to define and manage Databricks resources, such as jobs, pipelines, and models, as code.
- **Template-based Project Creation**: Bundles can be created from templates, enabling you to set organizational standards for new projects.

## Python/PySpark Examples

A typical Databricks Asset Bundle project structure for a Python project might look like this:

```text
.my-project/
├── databricks.yml
├── src/
│   ├── __init__.py
│   ├── main.py
│   └── utils.py
├── resources/
│   ├── job.yml
│   └── pipeline.yml
└── tests/
    ├── __init__.py
    └── test_main.py
```

### `databricks.yml`

This is the main configuration file for the bundle. It defines the bundle's name, artifacts, resources, and targets.

### Advanced example `databricks.yml`

```yaml
bundle:
  name: my-project
  description: Example advanced Databricks Asset Bundle with jobs, pipelines, permissions, and targets.

variables:
  env:
    description: Deployment environment
    default: dev
  schema_name:
    description: Unity Catalog schema (per target override)
    default: my_project_dev
  catalog:
    description: Unity Catalog catalog
    default: main

artifacts:
  wheel:
    type: wheel
    path: .
    build: python -m build --wheel
    python: "3.11"

resources:
  clusters:
    adhoc_cluster:
      name: my-project-adhoc
      spark_version: 15.3.x-scala2.12
      node_type_id: Standard_DS3_v2
      autoscale:
        min_workers: 1
        max_workers: 4
      spark_conf:
        spark.sql.shuffle.partitions: "200"
      custom_tags:
        project: ${bundle.name}
        env: ${var.env}

  jobs:
    etl_job:
      name: ${bundle.name}-etl-${var.env}
      tags:
        project: ${bundle.name}
        env: ${var.env}
      schedule:
        quartz_cron_expression: "0 0 * * * ?"   # hourly
        timezone_id: UTC
        pause_status: UNPAUSED
      email_notifications:
        on_failure:
          - platform-data@company.com
      timeout_seconds: 7200
      max_concurrent_runs: 1
      tasks:
        - task_key: ingest
          job_cluster_key: small_job_cluster
          python_wheel_task:
            package_name: my_project
            entry_point: ingest
            parameters:
              - "--env"
              - "${var.env}"
              - "--time"
              - "{{workflow.time}}"
          libraries:
            - whl: ${artifacts.wheel.path}/dist/*.whl
          environment_variables:
            ENV: ${var.env}
            SCHEMA: ${var.schema_name}
        - task_key: transform
          depends_on:
            - task_key: ingest
          job_cluster_key: small_job_cluster
          python_wheel_task:
            package_name: my_project
            entry_point: transform
          libraries:
            - whl: ${artifacts.wheel.path}/dist/*.whl
        - task_key: quality_checks
          depends_on:
            - task_key: transform
          job_cluster_key: small_job_cluster
          python_wheel_task:
            package_name: my_project
            entry_point: quality
          timeout_seconds: 900
        - task_key: publish_metrics
          depends_on:
            - task_key: quality_checks
          python_wheel_task:
            package_name: my_project
            entry_point: publish
          existing_cluster_id: ${resources.clusters.adhoc_cluster.id}
      job_clusters:
        - job_cluster_key: small_job_cluster
          new_cluster:
            spark_version: 15.3.x-scala2.12
            node_type_id: Standard_DS3_v2
            autoscale:
              min_workers: 2
              max_workers: 6
            spark_env_vars:
              PYSPARK_PYTHON: /databricks/python3/bin/python
            custom_tags:
              job: etl
              env: ${var.env}
      permissions:
        - level: CAN_MANAGE
          group_name: data-engineers
        - level: CAN_VIEW
          group_name: data-analysts

  pipelines:
    dlt_pipeline:
      name: ${bundle.name}-dlt-${var.env}
      catalog: ${var.catalog}
      target: ${var.schema_name}
      channel: CURRENT
      development: ${var.env != "prod"}
      photon: true
      continuous: false
      libraries:
        - notebook:
            path: ./src/dlt/bronze_notebook
        - notebook:
            path: ./src/dlt/silver_notebook
        - whl: ${artifacts.wheel.path}/dist/*.whl
      configuration:
        bundle.name: ${bundle.name}
        bundle.env: ${var.env}
      permissions:
        - level: CAN_MANAGE
          group_name: data-engineers
        - level: CAN_VIEW
          group_name: data-analysts

  registered_models:
    metrics_model:
      name: ${bundle.name}_metrics_${var.env}
      permissions:
        - level: CAN_MANAGE
          group_name: ml-engineers
        - level: CAN_READ
          group_name: data-analysts

targets:
  dev:
    mode: development
    default: true
    workspace:
      root_path: /Workspace/Users/dev@databricks.com/${bundle.name}
    variables:
      schema_name: my_project_dev
    run_as:
      user_name: dev@databricks.com

  prod:
    mode: production
    workspace:
      root_path: /Workspace/Users/prod@databricks.com/${bundle.name}
    variables:
      env: prod
      schema_name: my_project_prod
    run_as:
      service_principal_name: spn-prod-my-project
```

### `src/main.py`

This is the main entry point for the project. It contains the business logic of the application.

```python
from pyspark.sql import SparkSession
from utils import get_greeting

def main():
    spark = SparkSession.builder.appName("my-project").getOrCreate()
    greeting = get_greeting()
    print(greeting)

if __name__ == "__main__":
    main()
```

### `src/utils.py`

This file contains utility functions that can be used across the project.

```python
def get_greeting():
    return "Hello, World!"
```

### `tests/test_main.py`

This file contains unit tests for the project.

```python
import unittest
from src.utils import get_greeting

class TestMain(unittest.TestCase):
    def test_get_greeting(self):
        self.assertEqual(get_greeting(), "Hello, World!")

if __name__ == "__main__":
    unittest.main()
```

## Best Practices

- **Version Control**: Always maintain a versioned history of your code and infrastructure to facilitate rollback and compliance needs.
- **Custom Templates**: Create custom bundle templates to enforce organizational standards, including default permissions, service principals, and CI/CD configurations.
- **CI/CD Integration**: Integrate your bundles with CI/CD pipelines to automate testing and deployment.
- **Authentication**: Use OAuth user-to-machine (U2M) authentication for secure access to your Databricks workspaces.
- **Regular Updates**: Keep your Databricks CLI updated to the latest version to take advantage of new bundle features.

## Gotchas

- **Workspace Files**: Ensure that workspace files are enabled in your Databricks workspace. This is enabled by default in Databricks Runtime 11.3 LTS and above.
- **CLI Version**: Make sure you have the correct version of the Databricks CLI installed (v0.218.0 or above).
- **Authentication**: Incorrectly configured authentication can lead to deployment failures. Double-check your authentication settings.
- **Relative Paths**: Be careful with relative paths in your `databricks.yml` file. Paths are relative to the location of the `databricks.yml` file.

## Mock Questions

1. **What is the primary purpose of the `databricks.yml` file in a Databricks Asset Bundle?**

    a.  To define the business logic of the application.  
    b.  To define the bundle's name, artifacts, resources, and targets.  
    c.  To define the unit tests for the project.  
    d.  To define the project's dependencies.

2. **Which of the following is a best practice for managing Databricks Asset Bundles?**

    a.  Storing secrets directly in the `databricks.yml` file.  
    b.  Using a single bundle for all projects in your organization.  
    c.  Integrating bundles with CI/CD pipelines for automated testing and deployment.  
    d.  Manually deploying bundles to production environments.

3. **What is a common gotcha when working with Databricks Asset Bundles?**

    a.  Using the latest version of the Databricks CLI.  
    b.  Incorrectly configured authentication.  
    c.  Using version control for your bundle files.  
    d.  Enabling workspace files in your Databricks workspace.

## Answers

1. b
2. c
3. b

## References

[1] [What are Databricks Asset Bundles?](https://docs.databricks.com/en/dev-tools/bundles/index.html)
