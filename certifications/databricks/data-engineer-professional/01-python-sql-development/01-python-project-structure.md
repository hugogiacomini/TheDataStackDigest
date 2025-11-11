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

## Scalable Python Project Structure for DABs

For production-ready, scalable projects, consider adopting a more comprehensive structure that supports modularity, testing, and enterprise requirements:

```text
.enterprise-project/
├── databricks.yml
├── setup.py
├── requirements.txt
├── pyproject.toml
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── src/
│   ├── __init__.py
│   ├── common/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── logging_utils.py
│   │   └── spark_utils.py
│   ├── data_processing/
│   │   ├── __init__.py
│   │   ├── extractors.py
│   │   ├── transformers.py
│   │   └── loaders.py
│   ├── ml_models/
│   │   ├── __init__.py
│   │   ├── feature_engineering.py
│   │   └── model_training.py
│   └── udfs/
│       ├── __init__.py
│       ├── string_functions.py
│       └── date_functions.py
├── resources/
│   ├── jobs/
│   │   ├── data_pipeline_job.yml
│   │   └── ml_training_job.yml
│   ├── pipelines/
│   │   ├── bronze_to_silver.yml
│   │   └── silver_to_gold.yml
│   └── experiments/
│       └── ml_experiment.yml
├── tests/
│   ├── __init__.py
│   ├── unit/
│   │   ├── test_extractors.py
│   │   ├── test_transformers.py
│   │   └── test_udfs.py
│   ├── integration/
│   │   ├── test_pipeline.py
│   │   └── test_e2e.py
│   └── fixtures/
│       └── sample_data.json
├── configs/
│   ├── dev.yml
│   ├── staging.yml
│   └── prod.yml
└── docs/
    ├── README.md
    └── deployment_guide.md
```

### Advanced `databricks.yml` Configuration

```yaml
bundle:
  name: enterprise-project
  git:
    branch: ${var.git_branch}
    
include:
  - configs/*.yml

variables:
  catalog:
    description: Unity Catalog name
    default: dev_catalog
  schema:
    description: Schema name  
    default: default
  git_branch:
    description: Current git branch
    lookup:
      command: git rev-parse --abbrev-ref HEAD

artifacts:
  enterprise_project_wheel:
    type: whl
    path: .
    build: pip wheel . --no-deps --wheel-dir dist

resources:
  jobs:
    data_pipeline:
      name: ${bundle.name}-data-pipeline-${var.git_branch}
      job_clusters:
        - job_cluster_key: main_cluster
          new_cluster:
            spark_version: 13.3.x-scala2.12
            node_type_id: i3.xlarge
            autoscale:
              min_workers: 2
              max_workers: 8
            spark_conf:
              "spark.databricks.adaptive.coalescePartitions.enabled": "true"
              "spark.databricks.adaptive.coalescePartitions.parallelismFirst": "false"
      libraries:
        - whl: ../dist/*.whl
      tasks:
        - task_key: bronze_ingestion
          job_cluster_key: main_cluster
          python_wheel_task:
            package_name: enterprise_project
            entry_point: bronze_pipeline
          depends_on: []
        - task_key: silver_transformation
          job_cluster_key: main_cluster  
          python_wheel_task:
            package_name: enterprise_project
            entry_point: silver_pipeline
          depends_on:
            - task_key: bronze_ingestion

  pipelines:
    medallion_pipeline:
      name: ${bundle.name}-medallion-${var.git_branch}
      target: ${var.catalog}.${var.schema}
      libraries:
        - whl: ../dist/*.whl
      configuration:
        bundle.sourcePath: ${bundle.target.workspace.root_path}
      development: true

targets:
  dev:
    mode: development
    default: true
    workspace:
      host: https://your-workspace.databricks.com
    variables:
      catalog: dev_catalog
      schema: ${workspace.current_user.short_name}
      
  staging:
    mode: production
    workspace:
      host: https://your-workspace.databricks.com
    variables:
      catalog: staging_catalog
      schema: default
      
  prod:
    mode: production
    workspace:
      host: https://your-workspace.databricks.com  
    variables:
      catalog: prod_catalog
      schema: default
    run_as:
      service_principal_name: prod-service-principal
```

## Library Dependency Management

Managing external libraries and dependencies is crucial for reproducible and reliable Databricks deployments. This section covers various approaches to handle PyPI packages, local wheels, and source archives.

### Package Installation Methods

#### 1. PyPI Package Installation

**Via Cluster Libraries (UI/API):**
```python
# Install via Databricks CLI
databricks libraries install --cluster-id $CLUSTER_ID --pypi-package pandas==1.5.3
databricks libraries install --cluster-id $CLUSTER_ID --pypi-package requests>=2.28.0
```

**Via Notebook Magic Commands:**
```python
%pip install pandas==1.5.3
%pip install --upgrade requests
%pip install git+https://github.com/user/repo.git@v1.0.0
```

**Via Job Configuration:**
```yaml
# In databricks.yml
resources:
  jobs:
    data_processing:
      libraries:
        - pypi:
            package: pandas==1.5.3
        - pypi:
            package: scikit-learn>=1.0.0
        - pypi:
            package: custom-package
            repo: https://private-pypi.company.com/simple/
```

#### 2. Custom Wheel Packages

**Building and Installing Local Wheels:**

Create a `setup.py` for your project:
```python
from setuptools import setup, find_packages

setup(
    name="enterprise-project",
    version="1.0.0",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    install_requires=[
        "pandas>=1.5.0",
        "pyarrow>=10.0.0",
        "pydantic>=1.10.0",
    ],
    extras_require={
        "dev": ["pytest>=7.0.0", "black>=22.0.0", "flake8>=4.0.0"],
        "ml": ["scikit-learn>=1.0.0", "xgboost>=1.6.0"],
    },
    python_requires=">=3.8",
)
```

**Build and install wheel:**
```bash
# Build wheel locally
pip wheel . --no-deps --wheel-dir dist

# Install in Databricks job
libraries:
  - whl: ../dist/enterprise_project-1.0.0-py3-none-any.whl
```

#### 3. Source Archive Installation

**Via Job/Pipeline Configuration:**
```yaml
libraries:
  - egg: s3://my-bucket/packages/custom-package.tar.gz
  - jar: s3://my-bucket/jars/custom-jar.jar
```

### Dependency Management Best Practices

#### 1. Requirements Management

**`requirements.txt` (Production):**
```text
# Core dependencies with exact versions
pandas==1.5.3
pyarrow==10.0.1
pyspark==3.4.1

# Development dependencies
pytest==7.2.0
black==22.10.0
flake8==5.0.4
```

**`requirements-dev.txt` (Development):**
```text
-r requirements.txt

# Additional dev tools
jupyter==1.0.0
databricks-cli==0.18.0
databricks-connect==13.3.0
```

#### 2. Conda Environment File

**`environment.yml`:**
```yaml
name: enterprise-project
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.9
  - pandas=1.5.3
  - pyarrow=10.0.1
  - pip
  - pip:
    - databricks-connect==13.3.0
    - custom-internal-package
```

#### 3. Poetry Configuration

**`pyproject.toml`:**
```toml
[tool.poetry]
name = "enterprise-project"
version = "1.0.0"
description = "Scalable Databricks project"

[tool.poetry.dependencies]
python = "^3.8"
pandas = "^1.5.0"
pyarrow = "^10.0.0"
pydantic = "^1.10.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0.0"
black = "^22.0.0"
flake8 = "^4.0.0"

[tool.poetry.group.ml.dependencies]
scikit-learn = "^1.0.0"
xgboost = "^1.6.0"
```

### Troubleshooting Library Issues

#### Common Installation Problems

**1. Version Conflicts:**
```python
# Check installed packages
%pip list
%pip show pandas

# Force reinstall specific version
%pip install --force-reinstall pandas==1.5.3

# Install with no dependencies to avoid conflicts
%pip install --no-deps custom-package
```

**2. Private Package Repositories:**
```python
# Configure pip for private PyPI
%pip config set global.extra-index-url https://private-pypi.company.com/simple/
%pip config set global.trusted-host private-pypi.company.com

# Install with authentication
%pip install custom-package --index-url https://user:token@private-pypi.company.com/simple/
```

**3. Conda/Pip Conflicts:**
```python
# Use conda for system packages, pip for Python packages
%conda install -c conda-forge pyarrow=10.0.1
%pip install custom-python-package
```

#### Environment Isolation

**Per-Job Library Management:**
```yaml
# Different library sets per job
resources:
  jobs:
    ml_training:
      libraries:
        - pypi:
            package: scikit-learn==1.2.0
        - pypi:
            package: xgboost==1.7.0
    
    data_processing:
      libraries:
        - pypi:
            package: pandas==1.5.3
        - pypi:
            package: pyarrow==10.0.1
```

**Using Init Scripts for Complex Dependencies:**
```bash
#!/bin/bash
# cluster-init.sh

# Install system dependencies
apt-get update
apt-get install -y build-essential

# Configure private repositories
echo "extra-index-url = https://private-pypi.company.com/simple/" >> /etc/pip.conf

# Install specific versions
/databricks/python/bin/pip install pandas==1.5.3
```

## User-Defined Functions (UDFs)

User-Defined Functions allow you to extend Spark SQL with custom Python logic. This section covers Pandas UDFs and Python UDFs for optimal performance and functionality.

### Python UDFs

**Basic Python UDF:**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType, IntegerType

spark = SparkSession.builder.getOrCreate()

# Simple string transformation UDF
@udf(returnType=StringType())
def clean_text(text):
    if text is None:
        return None
    return text.strip().lower().replace(" ", "_")

# Numeric calculation UDF
@udf(returnType=IntegerType())
def calculate_age(birth_year):
    from datetime import datetime
    if birth_year is None:
        return None
    return datetime.now().year - birth_year

# Usage in DataFrame
df = spark.createDataFrame([
    ("  John Doe  ", 1990),
    ("Jane Smith", 1985),
    (None, 1995)
], ["name", "birth_year"])

result = df.select(
    clean_text(df.name).alias("cleaned_name"),
    calculate_age(df.birth_year).alias("age")
)
```

### Pandas UDFs (Vectorized UDFs)

Pandas UDFs provide significantly better performance for complex operations by leveraging Apache Arrow for data transfer.

#### Pandas UDF Types

**1. Scalar Pandas UDF:**
```python
import pandas as pd
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import StringType, IntegerType, DoubleType

# String processing UDF
@pandas_udf(returnType=StringType())
def vectorized_text_clean(series: pd.Series) -> pd.Series:
    return series.str.strip().str.lower().str.replace(" ", "_")

# Numeric calculation UDF
@pandas_udf(returnType=DoubleType())
def calculate_bmi(height_series: pd.Series, weight_series: pd.Series) -> pd.Series:
    # height in meters, weight in kg
    return weight_series / (height_series ** 2)

# Complex business logic UDF
@pandas_udf(returnType=StringType())
def categorize_customer(age_series: pd.Series, income_series: pd.Series) -> pd.Series:
    def categorize_single(age, income):
        if pd.isna(age) or pd.isna(income):
            return "Unknown"
        elif age < 25:
            return "Young"
        elif age >= 65:
            return "Senior"
        elif income > 100000:
            return "High Value"
        else:
            return "Standard"
    
    return pd.Series([categorize_single(age, income) 
                     for age, income in zip(age_series, income_series)])
```

**2. Grouped Map Pandas UDF:**
```python
from pyspark.sql.types import StructType, StructField
from pyspark.sql import DataFrame

# Define return schema
return_schema = StructType([
    StructField("group_id", StringType(), True),
    StructField("avg_value", DoubleType(), True),
    StructField("std_value", DoubleType(), True),
    StructField("percentile_95", DoubleType(), True)
])

@pandas_udf(returnType=return_schema, functionType=PandasUDFType.GROUPED_MAP)
def calculate_group_statistics(pdf: pd.DataFrame) -> pd.DataFrame:
    group_id = pdf["group_id"].iloc[0]
    values = pdf["value"]
    
    return pd.DataFrame([{
        "group_id": group_id,
        "avg_value": values.mean(),
        "std_value": values.std(),
        "percentile_95": values.quantile(0.95)
    }])

# Usage with groupBy
result = df.groupBy("group_id").apply(calculate_group_statistics)
```

**3. Window Pandas UDF:**
```python
from pyspark.sql.window import Window

# Define window specification
window_spec = Window.partitionBy("category").orderBy("date")

@pandas_udf(returnType=DoubleType())
def calculate_moving_average(series: pd.Series) -> pd.Series:
    return series.rolling(window=7, min_periods=1).mean()

# Apply window function
df_with_ma = df.withColumn(
    "moving_avg", 
    calculate_moving_average("value").over(window_spec)
)
```

### Advanced UDF Patterns

#### 1. UDF with External Dependencies

```python
# src/udfs/advanced_functions.py
import pandas as pd
import numpy as np
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import DoubleType, ArrayType

# UDF using external ML library
@pandas_udf(returnType=ArrayType(DoubleType()))
def predict_with_model(features_series: pd.Series) -> pd.Series:
    # Import inside UDF to ensure availability on workers
    from sklearn.externals import joblib
    import numpy as np
    
    # Load pre-trained model (could be from DBFS, S3, etc.)
    model = joblib.load("/dbfs/models/trained_model.pkl")
    
    # Convert features and predict
    features_array = np.array([eval(features) for features in features_series])
    predictions = model.predict(features_array)
    
    return pd.Series(predictions.tolist())

# UDF with complex data processing
@pandas_udf(returnType=DoubleType())
def calculate_text_similarity(text1_series: pd.Series, text2_series: pd.Series) -> pd.Series:
    from sklearn.feature_extraction.text import TfidfVectorizer
    from sklearn.metrics.pairwise import cosine_similarity
    import numpy as np
    
    similarities = []
    
    for text1, text2 in zip(text1_series, text2_series):
        if pd.isna(text1) or pd.isna(text2):
            similarities.append(None)
            continue
            
        vectorizer = TfidfVectorizer()
        try:
            tfidf_matrix = vectorizer.fit_transform([text1, text2])
            similarity = cosine_similarity(tfidf_matrix[0:1], tfidf_matrix[1:2])[0][0]
            similarities.append(float(similarity))
        except:
            similarities.append(0.0)
    
    return pd.Series(similarities)
```

#### 2. Performance Optimized UDFs

```python
# src/udfs/optimized_functions.py
from typing import Iterator
import pandas as pd
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import StringType

# Batch processing UDF for better memory management
@pandas_udf(returnType=StringType())
def batch_process_text(text_series: pd.Series) -> pd.Series:
    """
    Process text in batches to manage memory usage
    """
    batch_size = 1000
    processed_texts = []
    
    for i in range(0, len(text_series), batch_size):
        batch = text_series.iloc[i:i+batch_size]
        
        # Complex processing on batch
        batch_processed = (batch
                          .str.lower()
                          .str.replace(r'[^\w\s]', '', regex=True)
                          .str.strip())
        
        processed_texts.extend(batch_processed.tolist())
    
    return pd.Series(processed_texts)

# Cached computation UDF
_EXPENSIVE_COMPUTATION_CACHE = {}

@pandas_udf(returnType=DoubleType())
def cached_expensive_computation(input_series: pd.Series) -> pd.Series:
    """
    Cache expensive computations to avoid recomputation
    """
    results = []
    
    for value in input_series:
        if value in _EXPENSIVE_COMPUTATION_CACHE:
            results.append(_EXPENSIVE_COMPUTATION_CACHE[value])
        else:
            # Expensive computation
            result = complex_calculation(value)
            _EXPENSIVE_COMPUTATION_CACHE[value] = result
            results.append(result)
    
    return pd.Series(results)

def complex_calculation(value):
    # Simulate expensive computation
    import time
    import math
    time.sleep(0.001)  # Simulate processing time
    return math.sqrt(value) if value >= 0 else 0
```

#### 3. UDF Testing and Validation

```python
# tests/unit/test_udfs.py
import pytest
import pandas as pd
from pyspark.sql import SparkSession
from pyspark.sql.types import StringType, DoubleType

from src.udfs.string_functions import clean_text, vectorized_text_clean
from src.udfs.advanced_functions import calculate_bmi

class TestUDFs:
    @classmethod
    def setup_class(cls):
        cls.spark = SparkSession.builder.master("local[*]").getOrCreate()
    
    def test_clean_text_udf(self):
        # Test data
        test_data = [("  John Doe  ",), ("Jane Smith",), (None,)]
        df = self.spark.createDataFrame(test_data, ["name"])
        
        # Apply UDF
        result = df.select(clean_text(df.name).alias("cleaned")).collect()
        
        # Assertions
        assert result[0]["cleaned"] == "john_doe"
        assert result[1]["cleaned"] == "jane_smith"
        assert result[2]["cleaned"] is None
    
    def test_vectorized_text_clean(self):
        # Test pandas UDF with pandas Series
        test_series = pd.Series(["  John Doe  ", "Jane Smith", None])
        result = vectorized_text_clean(test_series)
        
        expected = pd.Series(["john_doe", "jane_smith", None])
        pd.testing.assert_series_equal(result, expected)
    
    def test_calculate_bmi_udf(self):
        # Test BMI calculation
        height_data = [1.75, 1.80, 1.65]
        weight_data = [70, 85, 55]
        
        df = self.spark.createDataFrame(
            list(zip(height_data, weight_data)), 
            ["height", "weight"]
        )
        
        result = df.select(
            calculate_bmi(df.height, df.weight).alias("bmi")
        ).collect()
        
        # BMI = weight / height^2
        assert abs(result[0]["bmi"] - (70 / (1.75 ** 2))) < 0.01
        assert abs(result[1]["bmi"] - (85 / (1.80 ** 2))) < 0.01
        assert abs(result[2]["bmi"] - (55 / (1.65 ** 2))) < 0.01
```

### UDF Best Practices

1. **Use Pandas UDFs for Performance**: Prefer Pandas UDFs over Python UDFs for better performance through vectorization
2. **Handle Null Values**: Always handle null/None values in your UDF logic
3. **Import Libraries Inside UDFs**: Import external libraries inside the UDF function to ensure availability on worker nodes
4. **Minimize Data Transfer**: Keep UDF logic lean and minimize data movement
5. **Cache Expensive Computations**: Use caching for repeated expensive operations
6. **Test Thoroughly**: Write comprehensive unit tests for your UDFs
7. **Use Appropriate Return Types**: Specify correct return types to avoid runtime errors
8. **Batch Processing**: For large datasets, consider batch processing within UDFs

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
- **Modular Architecture**: Structure your code into logical modules (extractors, transformers, loaders) for better maintainability and reusability.
- **Dependency Management**: Use exact version pinning for production dependencies and separate development/testing dependencies.
- **Environment Isolation**: Use different library configurations for different environments (dev/staging/prod) to ensure consistency.
- **UDF Performance**: Prefer Pandas UDFs over Python UDFs for better performance, and always handle null values properly.
- **Testing Strategy**: Implement comprehensive unit tests for UDFs and integration tests for data pipelines.
- **Documentation**: Maintain clear documentation for your project structure, dependencies, and deployment procedures.

## Gotchas

- **Workspace Files**: Ensure that workspace files are enabled in your Databricks workspace. This is enabled by default in Databricks Runtime 11.3 LTS and above.
- **CLI Version**: Make sure you have the correct version of the Databricks CLI installed (v0.218.0 or above).
- **Authentication**: Incorrectly configured authentication can lead to deployment failures. Double-check your authentication settings.
- **Relative Paths**: Be careful with relative paths in your `databricks.yml` file. Paths are relative to the location of the `databricks.yml` file.
- **Library Conflicts**: Version conflicts between dependencies can cause runtime errors. Use virtual environments and exact version pinning.
- **UDF Serialization**: Large objects or complex data structures in UDFs may cause serialization issues. Keep UDF logic lightweight.
- **Pandas UDF Types**: Ensure you're using the correct Pandas UDF type (scalar, grouped map, window) for your use case.
- **Import Statements**: Import external libraries inside UDF functions to ensure availability on worker nodes.
- **Memory Management**: Large datasets in Pandas UDFs can cause out-of-memory errors. Consider batch processing for large operations.
- **Null Handling**: Always handle null/None values in UDFs to prevent runtime exceptions.

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

4. **Which method provides the best performance for complex data transformations in Spark UDFs?**

    a.  Python UDFs with complex logic  
    b.  Pandas UDFs leveraging vectorization  
    c.  SQL-only transformations without UDFs  
    d.  Java-based UDFs

5. **What is the recommended approach for managing external library dependencies in a scalable Databricks project?**

    a.  Install packages directly in notebooks using %pip install  
    b.  Use exact version pinning in requirements.txt and separate dev/prod dependencies  
    c.  Install all packages at the cluster level only  
    d.  Use conda environments exclusively

6. **When developing Pandas UDFs, what is the most important consideration for null value handling?**

    a.  Pandas automatically handles all null values  
    b.  Null values should be filtered out before applying the UDF  
    c.  Always explicitly handle None/null values in the UDF logic  
    d.  Use spark.sql.adaptive.coalescePartitions.enabled for null handling

7. **What is the benefit of using a modular project structure with separate extractors, transformers, and loaders?**

    a.  It reduces the overall codebase size  
    b.  It improves maintainability, reusability, and testing capabilities  
    c.  It automatically optimizes Spark performance  
    d.  It eliminates the need for version control

**Answers:**

1. b
2. c
3. b
4. b
5. b
6. c
7. b

## References

[1] [What are Databricks Asset Bundles?](https://docs.databricks.com/en/dev-tools/bundles/index.html)
