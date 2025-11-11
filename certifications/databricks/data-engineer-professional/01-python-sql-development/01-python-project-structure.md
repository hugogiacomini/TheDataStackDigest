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

```
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

```yaml
bundle:
  name: my-project

artifacts:
  - path: src

resources:
  jobs:
    my_job:
      name: My Job
      tasks:
        - task_key: my_task
          python_wheel_task:
            package_name: my-project
            entry_point: main

  pipelines:
    my_pipeline:
      name: My Pipeline
      target: my-project
      libraries:
        - notebook:
            path: "../src/main.py"

targets:
  dev:
    mode: development
    default: true
  prod:
    mode: production
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

**Answers:**

1. b
2. c
3. b

## References

[1] [What are Databricks Asset Bundles?](https://docs.databricks.com/en/dev-tools/bundles/index.html)
