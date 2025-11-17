# Feature Store and Model Registry

## Overview

Manage ML features and models: Feature Store for training/serving, Model Registry for versioning, staging, and approvals.

## Prerequisites

- Databricks ML Runtime
- Unity Catalog for feature tables and model registry

## Concepts

- Feature Store: Centralized repo for feature definitions; supports online/offline serving
- Model Registry: Versioned model artifacts with stage transitions (None → Staging → Production)
- Serving: Batch vs real-time inference with Feature Store lookups

## Hands-on Walkthrough

### Create Feature Table (Python)

```python
from databricks.feature_store import FeatureStoreClient

fs = FeatureStoreClient()

# Define feature table
feature_df = spark.sql("""
  SELECT customer_id,
         AVG(order_amount) AS avg_order_amount,
         COUNT(*) AS order_count
  FROM prod.retail.orders_silver
  GROUP BY customer_id
""")

fs.create_table(
    name="prod.features.customer_features",
    primary_keys=["customer_id"],
    df=feature_df,
    description="Customer aggregation features"
)
```

### Train Model with Feature Store (Python)

```python
from databricks.feature_store import FeatureLookup

# Define feature lookups
feature_lookups = [
    FeatureLookup(
        table_name="prod.features.customer_features",
        lookup_key="customer_id"
    )
]

# Create training set
training_set = fs.create_training_set(
    df=spark.table("prod.ml.training_labels"),
    feature_lookups=feature_lookups,
    label="churn_label"
)

training_df = training_set.load_df()

# Train model (e.g., sklearn)
from sklearn.ensemble import RandomForestClassifier
model = RandomForestClassifier()
model.fit(training_df.drop("churn_label"), training_df["churn_label"])

# Log model with feature metadata
fs.log_model(
    model=model,
    artifact_path="model",
    flavor=mlflow.sklearn,
    training_set=training_set,
    registered_model_name="prod.models.churn_classifier"
)
```

### Transition Model Stage (Python)

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()

# Transition to Staging
client.transition_model_version_stage(
    name="prod.models.churn_classifier",
    version=1,
    stage="Staging"
)

# After validation, promote to Production
client.transition_model_version_stage(
    name="prod.models.churn_classifier",
    version=1,
    stage="Production"
)
```

### Batch Scoring with Feature Store (Python)

```python
batch_df = spark.table("prod.ml.scoring_batch")

predictions = fs.score_batch(
    model_uri="models:/prod.models.churn_classifier/Production",
    df=batch_df
)

predictions.write.mode("overwrite").saveAsTable("prod.ml.churn_predictions")
```

## Production Considerations

- Feature freshness: Schedule feature table updates; monitor staleness.
- Online serving: Use Feature Store online stores for low-latency lookups.
- Model governance: Enforce approval workflows before Production transitions.
- Versioning: Tag models with metadata (training date, metrics, author).

## Troubleshooting

- Feature lookup failures: Verify primary keys exist in scoring data.
- Model version conflicts: Use explicit version URIs or aliases.
- Stale features: Automate feature table refreshes; alert on lag.

## Sample Questions

1. What is the purpose of the Feature Store?  
2. How do you transition a model to Production?  
3. What are the benefits of logging models with Feature Store metadata?  
4. When should online vs offline feature serving be used?  
5. How do you enforce approval workflows for model deployment?

## Answers

1. Centralized feature repository for consistent training/serving and reusability.  
2. Use `MlflowClient.transition_model_version_stage()` with stage="Production".  
3. Ensures consistent feature lookups during inference; tracks lineage.  
4. Online for real-time APIs; offline for batch scoring.  
5. Implement approval gates in CI/CD before transitioning to Production stage.

## References

- [Feature Store](https://docs.databricks.com/machine-learning/feature-store/index.html)
- [Model Registry](https://docs.databricks.com/machine-learning/manage-model-lifecycle/index.html)

---

Previous: [MLOps in Databricks](./01-mlops.md)  
Next: [Model Monitoring and Drift Management](./03-model-monitoring-drift-management.md)
