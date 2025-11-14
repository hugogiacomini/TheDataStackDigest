# Model Monitoring and Drift Management

## Overview

Detect and respond to model performance degradation: data drift, concept drift, monitoring via Lakehouse Monitoring, and retraining triggers.

## Prerequisites

- Deployed production models
- Access to inference logs and ground truth labels

## Concepts

- Data drift: Input feature distributions shift over time
- Concept drift: Relationship between features and target changes
- Lakehouse Monitoring: Automated drift detection on Delta tables
- Retraining triggers: Alert-based or scheduled model refresh

## Hands-on Walkthrough

### Enable Lakehouse Monitoring (Python)

```python
from databricks.lakehouse_monitoring import create_monitor

create_monitor(
    table_name="prod.ml.churn_predictions",
    profile_type="InferenceLog",
    model_id="prod.models.churn_classifier",
    prediction_col="prediction",
    label_col="actual_churn",  # Ground truth when available
    problem_type="classification",
    baseline_table="prod.ml.training_baseline"
)
```

### Query Drift Metrics (SQL)

```sql
SELECT window_start_time, feature_name, drift_score, drift_threshold
FROM prod.ml.churn_predictions_drift_metrics
WHERE drift_score > drift_threshold
ORDER BY window_start_time DESC, drift_score DESC
LIMIT 50;
```

### Custom Drift Detection (Python)

```python
from scipy.stats import ks_2samp
import pandas as pd

baseline_df = spark.table("prod.ml.training_baseline").toPandas()
current_df = spark.table("prod.ml.recent_predictions").toPandas()

for col in ["avg_order_amount", "order_count"]:
    stat, p_value = ks_2samp(baseline_df[col], current_df[col])
    if p_value < 0.05:
        print(f"Drift detected in {col}: KS statistic={stat:.4f}, p={p_value:.4f}")
```

### Trigger Retraining Alert (SQL)

```sql
CREATE ALERT model_drift_alert
SCHEDULE CRON '0 0 * * *'
AS
SELECT COUNT(*) AS drifted_features
FROM prod.ml.churn_predictions_drift_metrics
WHERE drift_score > drift_threshold
  AND window_start_time >= current_date() - INTERVAL 1 DAY
HAVING COUNT(*) > 3;
```

## Production Considerations

- Monitoring frequency: Align with model refresh cadence and data velocity.
- Ground truth latency: Labels may lag predictions; use proxy metrics initially.
- Alert thresholds: Tune to balance false positives and timely detection.
- Automated retraining: Integrate alerts with orchestration (Workflows) to trigger pipelines.

## Troubleshooting

- False drift alarms: Validate statistical tests; consider seasonal patterns.
- Missing ground truth: Use unsupervised drift detection until labels arrive.
- Monitor overhead: Sample high-volume inference logs to reduce compute cost.

## Sample Questions

1. What is the difference between data drift and concept drift?  
2. How does Lakehouse Monitoring detect drift?  
3. When should model retraining be triggered?  
4. What statistical test is commonly used for drift detection?  
5. How do you handle delayed ground truth labels?

## Answers

1. Data drift: feature distribution changes; concept drift: feature-target relationship changes.  
2. Compares inference data distributions to baseline using statistical tests.  
3. When drift exceeds thresholds or performance metrics degrade.  
4. Kolmogorov-Smirnov (KS) test or Population Stability Index (PSI).  
5. Use proxy metrics (prediction confidence, feature drift) until labels are available.

## References

- [Lakehouse Monitoring](https://docs.databricks.com/machine-learning/lakehouse-monitoring/index.html)
- [Model drift detection](https://www.databricks.com/blog/2021/10/21/monitoring-ml-models-at-scale.html)

---

Previous: [Feature Store and Model Registry](./02-feature-store-and-model-registry.md)
