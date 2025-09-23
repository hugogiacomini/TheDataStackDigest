# Getting Started with Apache Airflow for Data Engineering

## Overview

Apache Airflow is an open-source platform for developing, scheduling, and monitoring workflows. It's particularly valuable for data engineering teams who need to orchestrate complex data pipelines with dependencies, error handling, and scheduling capabilities.

This guide covers the fundamentals of setting up and using Airflow for data engineering workflows.

## Prerequisites

- Basic Python programming knowledge
- Understanding of data pipeline concepts
- Docker installed (for local development)
- Familiarity with command line operations

## What is Apache Airflow?

Airflow represents workflows as Directed Acyclic Graphs (DAGs) of tasks. Each DAG defines:
- **Tasks**: Individual units of work
- **Dependencies**: Relationships between tasks
- **Schedule**: When and how often the workflow runs
- **Configuration**: Parameters and settings

### Key Concepts

- **DAG (Directed Acyclic Graph)**: A collection of tasks with defined dependencies
- **Task**: A single unit of work (Python function, bash command, etc.)
- **Operator**: Defines what actually gets executed (PythonOperator, BashOperator, etc.)
- **Scheduler**: Orchestrates the execution of DAGs
- **Executor**: Determines how tasks are executed (sequential, parallel, distributed)
- **Webserver**: Provides the UI for monitoring and managing workflows

## Quick Start with Docker

### 1. Set up Airflow with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres_db_volume:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

  airflow-webserver:
    image: apache/airflow:2.7.1
    depends_on:
      - postgres
      - redis
    environment:
      AIRFLOW__CORE__EXECUTOR: CeleryExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
      AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    ports:
      - "8080:8080"
    command: webserver

  airflow-scheduler:
    image: apache/airflow:2.7.1
    depends_on:
      - postgres
      - redis
    environment:
      AIRFLOW__CORE__EXECUTOR: CeleryExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
      AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    command: scheduler

volumes:
  postgres_db_volume:
```

### 2. Initialize and Start Airflow

```bash
# Create directories
mkdir -p dags logs plugins

# Initialize the database
docker-compose up airflow-init

# Start Airflow
docker-compose up -d
```

### 3. Access the Web UI

Navigate to `http://localhost:8080` and log in with:
- Username: `airflow`
- Password: `airflow`

## Creating Your First DAG

Create a file `dags/sample_data_pipeline.py`:

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

# Default arguments for the DAG
default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Define the DAG
dag = DAG(
    'sample_data_pipeline',
    default_args=default_args,
    description='A sample data engineering pipeline',
    schedule_interval=timedelta(days=1),
    catchup=False,
)

def extract_data():
    """Extract data from source"""
    print("Extracting data from source...")
    # Your extraction logic here
    return "Data extracted successfully"

def transform_data():
    """Transform the extracted data"""
    print("Transforming data...")
    # Your transformation logic here
    return "Data transformed successfully"

def load_data():
    """Load data to destination"""
    print("Loading data to destination...")
    # Your loading logic here
    return "Data loaded successfully"

# Define tasks
extract_task = PythonOperator(
    task_id='extract_data',
    python_callable=extract_data,
    dag=dag,
)

transform_task = PythonOperator(
    task_id='transform_data',
    python_callable=transform_data,
    dag=dag,
)

load_task = PythonOperator(
    task_id='load_data',
    python_callable=load_data,
    dag=dag,
)

# Data quality check
quality_check = BashOperator(
    task_id='data_quality_check',
    bash_command='echo "Running data quality checks..."',
    dag=dag,
)

# Define task dependencies
extract_task >> transform_task >> load_task >> quality_check
```

## Best Practices for Data Engineering

### 1. DAG Design Principles

- **Keep DAGs simple**: Each DAG should have a single responsibility
- **Use meaningful names**: Task and DAG IDs should be descriptive
- **Handle failures gracefully**: Implement proper error handling and retries
- **Make tasks idempotent**: Tasks should produce the same result when run multiple times

### 2. Configuration Management

```python
from airflow.models import Variable
from airflow.hooks.base import BaseHook

# Use Airflow Variables for configuration
database_conn = BaseHook.get_connection('my_database')
batch_size = Variable.get('batch_size', default_var=1000)
```

### 3. Testing Your DAGs

```python
import pytest
from airflow.models import DagBag

def test_dag_loaded():
    dag_bag = DagBag()
    dag = dag_bag.get_dag(dag_id='sample_data_pipeline')
    assert dag is not None
    assert len(dag.tasks) == 4
```

### 4. Monitoring and Alerting

- Set up email notifications for failures
- Use Airflow's built-in metrics
- Implement custom health checks
- Monitor resource usage

## Common Use Cases

### Data Lake ETL Pipeline
```python
# Extract from multiple sources
extract_customers = PythonOperator(...)
extract_orders = PythonOperator(...)

# Transform and combine
transform_customer_orders = PythonOperator(...)

# Load to data lake
load_to_s3 = S3FileTransferOperator(...)

# Set dependencies
[extract_customers, extract_orders] >> transform_customer_orders >> load_to_s3
```

### Data Warehouse Loading
```python
# Extract from operational databases
extract_from_postgres = PostgresOperator(...)

# Transform using dbt
dbt_transform = BashOperator(
    task_id='dbt_transform',
    bash_command='dbt run --models {{ ds }}',
)

# Data quality tests
dbt_test = BashOperator(
    task_id='dbt_test',
    bash_command='dbt test',
)

extract_from_postgres >> dbt_transform >> dbt_test
```

## Key Takeaways

- **Airflow excels at orchestration**: Use it to coordinate complex workflows, not for data processing itself
- **Design for maintainability**: Write clean, documented DAGs that your team can understand
- **Test thoroughly**: Test DAGs locally before deploying to production
- **Monitor actively**: Set up proper monitoring and alerting for production workflows
- **Scale appropriately**: Choose the right executor for your workload (Sequential, Local, Celery, Kubernetes)

## Resources

- [Apache Airflow Official Documentation](https://airflow.apache.org/docs/)
- [Airflow Best Practices Guide](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
- [Astronomer's Airflow Guides](https://docs.astronomer.io/)
- [Airflow GitHub Repository](https://github.com/apache/airflow)
- [Airflow Community](https://airflow.apache.org/community/)

## Author

*This article serves as a template for TheDataStackDigest contributors. Replace this section with your information when contributing.*

- **Author**: Data Engineering Community
- **Published**: January 2024
- **Last Updated**: January 2024

---

*This is a sample article demonstrating the structure and style for TheDataStackDigest content. Contributors should follow this format when adding new articles.*