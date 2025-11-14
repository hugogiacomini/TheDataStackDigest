# MLOps in Databricks

## Concept Definition

MLOps is a set of practices that combines Machine Learning, DevOps, and Data Engineering to automate and streamline the end-to-end machine learning lifecycle. In Databricks, MLOps is supported by a suite of tools that help you manage the entire ML lifecycle, from data preparation to model deployment and monitoring. [1]

## Key MLOps Tools in Databricks

- **MLflow**: An open-source platform for managing the end-to-end machine learning lifecycle. It includes components for tracking experiments, packaging code, and deploying models. [2]
- **Feature Store**: A centralized repository for storing, sharing, and discovering features for machine learning models. [3]
- **Model Serving**: A feature for deploying machine learning models as REST APIs.
- **Model Registry**: A centralized model store to manage the full lifecycle of ML models.

## Examples

### Tracking an Experiment with MLflow

```python
import mlflow

with mlflow.start_run():
    mlflow.log_param("my_param", "my_value")
    mlflow.log_metric("my_metric", 0.95)
    mlflow.sklearn.log_model(my_model, "my_model")
```

### Creating a Feature Table

```python
from databricks.feature_store import FeatureStoreClient

fs = FeatureStoreClient()

fs.create_table(
    name="my_feature_table",
    primary_keys=["my_key"],
    df=my_feature_df,
    description="My feature table"
)
```

## Best Practices

- **Use MLflow to Track Everything**: Use MLflow to track your experiments, parameters, metrics, and models. This will help you to reproduce your results and collaborate with others.
- **Use the Feature Store to Share and Reuse Features**: Use the Feature Store to share and reuse features across your organization. This will help to improve consistency and reduce redundant work.
- **Use Model Serving for Real-Time Inference**: Use Model Serving to deploy your models as REST APIs for real-time inference.
- **Version Your Models**: Use the Model Registry to version your models and manage their lifecycle.

## Gotchas

- **MLOps is a Culture, Not Just a Tool**: MLOps is not just about using a set of tools. It is a culture of collaboration and automation that needs to be adopted by your entire team.
- **Feature Engineering is Key**: The quality of your features is a key determinant of the performance of your models. Be sure to invest time in feature engineering.
- **Monitoring is Crucial**: Be sure to monitor your models in production to ensure they are performing as expected.

## Mock Questions

1. **Which Databricks tool is an open-source platform for managing the end-to-end machine learning lifecycle?**

    a. Feature Store  
    b. Model Serving  
    c. MLflow  
    d. Unity Catalog

2. **What is the primary purpose of the Feature Store?**

    a. To deploy models as REST APIs.  
    b. To track experiments.  
    c. To provide a centralized repository for storing, sharing, and discovering features.  
    d. To manage the lifecycle of models.

3. **What is a key benefit of using the Model Registry?**

    a. It allows you to version your models and manage their lifecycle.  
    b. It automatically trains your models.  
    c. It provides a centralized repository for storing your data.  
    d. It is the only way to deploy models in Databricks.

## Answers

1. c
2. c
3. a

## References

[1] [MLOps workflows on Databricks](https://docs.databricks.com/aws/en/machine-learning/mlops/mlops-workflow)
[2] [Managed MLflow](https://www.databricks.com/product/managed-mlflow)
[3] [Feature Store](https://www.databricks.com/product/feature-store)
