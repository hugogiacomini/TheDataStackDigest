# RAG Performance Evaluation with MLflow

## Overview

Evaluate RAG application performance using MLflow's evaluation framework: measure answer quality, faithfulness, relevance, and retrieval effectiveness.

## Key Metrics

| Metric | Definition | MLflow Function |
|--------|-----------|----------------|
| **Answer Relevance** | How well answer addresses question | `answer_relevance` |
| **Faithfulness** | Answer grounded in retrieved context | `faithfulness` |
| **Context Precision** | Relevant docs in top-K | `context_precision` |
| **Context Recall** | Ground truth info in retrieved docs | `context_recall` |
| **Answer Correctness** | Semantic similarity to ground truth | `answer_correctness` |

## Hands-on Examples

### Basic RAG Evaluation

```python
import mlflow
import pandas as pd

# Evaluation dataset
eval_data = pd.DataFrame({
    "question": [
        "What is Unity Catalog?",
        "How does Delta Lake ensure ACID compliance?"
    ],
    "ground_truth": [
        "Unity Catalog is a unified governance solution for data and AI assets.",
        "Delta Lake uses a transaction log with optimistic concurrency control."
    ]
})

# Evaluate RAG model
with mlflow.start_run():
    results = mlflow.evaluate(
        model="models:/rag_chatbot/production",
        data=eval_data,
        targets="ground_truth",
        model_type="question-answering",
        evaluators="default"
    )
    
    print(results.metrics)
    # {'answer_relevance': 0.87, 'faithfulness': 0.92, ...}
```

### Custom RAG Evaluation with RAGAS

```python
from ragas import evaluate
from ragas.metrics import (
    answer_relevancy,
    faithfulness,
    context_recall,
    context_precision
)
from datasets import Dataset

# Prepare evaluation data with contexts
eval_dataset = Dataset.from_dict({
    'question': ['What is Delta Lake?'],
    'answer': ['Delta Lake is an open-source storage layer...'],
    'contexts': [['Delta Lake provides ACID transactions...', 'Delta supports time travel...']],
    'ground_truth': ['Delta Lake is a storage framework for data lakes']
})

# Evaluate
results = evaluate(
    eval_dataset,
    metrics=[answer_relevancy, faithfulness, context_recall, context_precision]
)

print(results)
# {'answer_relevancy': 0.95, 'faithfulness': 0.88, ...}

# Log to MLflow
with mlflow.start_run():
    mlflow.log_metrics({
        f"ragas_{k}": v for k, v in results.items()
    })
```

### MLflow Evaluation with Custom Metrics

```python
import mlflow
from mlflow.models import make_metric

def citation_accuracy(predictions, targets, metrics):
    """Custom metric: check if citations are valid."""
    
    scores = []
    for pred, target in zip(predictions, targets):
        # Check if predicted sources are in ground truth sources
        pred_sources = set(pred.get('sources', []))
        true_sources = set(target.get('sources', []))
        
        if len(pred_sources) == 0:
            scores.append(0.0)
        else:
            accuracy = len(pred_sources & true_sources) / len(pred_sources)
            scores.append(accuracy)
    
    return {"citation_accuracy": sum(scores) / len(scores)}

citation_metric = make_metric(
    eval_fn=citation_accuracy,
    greater_is_better=True,
    name="citation_accuracy"
)

# Evaluate with custom metric
results = mlflow.evaluate(
    model="models:/rag_chatbot/production",
    data=eval_data,
    extra_metrics=[citation_metric]
)
```

### End-to-End Evaluation Pipeline

```python
class RAGEvaluator:
    def __init__(self, model_uri, eval_queries):
        self.model = mlflow.pyfunc.load_model(model_uri)
        self.eval_queries = eval_queries
    
    def generate_predictions(self):
        """Generate predictions for evaluation dataset."""
        
        predictions = []
        for query_data in self.eval_queries:
            pred = self.model.predict({"query": query_data["question"]})
            predictions.append({
                "question": query_data["question"],
                "answer": pred["answer"],
                "contexts": pred.get("retrieved_contexts", []),
                "sources": pred.get("sources", [])
            })
        
        return pd.DataFrame(predictions)
    
    def evaluate(self):
        """Run comprehensive evaluation."""
        
        pred_df = self.generate_predictions()
        
        # Prepare ground truth
        ground_truth_df = pd.DataFrame(self.eval_queries)
        eval_df = pred_df.merge(ground_truth_df, on="question")
        
        with mlflow.start_run():
            # MLflow evaluation
            mlflow_results = mlflow.evaluate(
                data=eval_df,
                model_type="question-answering",
                targets="ground_truth"
            )
            
            # RAGAS evaluation
            ragas_dataset = Dataset.from_pandas(eval_df[['question', 'answer', 'contexts', 'ground_truth']])
            ragas_results = evaluate(ragas_dataset, metrics=[answer_relevancy, faithfulness])
            
            # Combine results
            all_metrics = {
                **mlflow_results.metrics,
                **ragas_results
            }
            
            mlflow.log_metrics(all_metrics)
            
            # Log evaluation dataset
            mlflow.log_table(eval_df, "evaluation_results.json")
            
            return all_metrics

# Usage
evaluator = RAGEvaluator(
    model_uri="models:/rag_chatbot/production",
    eval_queries=[
        {"question": "What is Unity Catalog?", "ground_truth": "...", "true_sources": [...]},
        # ... more queries
    ]
)

metrics = evaluator.evaluate()
```

### Comparing Model Versions

```python
def compare_rag_versions(model_v1, model_v2, eval_data):
    """Compare two RAG model versions."""
    
    results = {}
    
    for version, model_uri in [("v1", model_v1), ("v2", model_v2)]:
        with mlflow.start_run(run_name=f"eval_{version}"):
            result = mlflow.evaluate(
                model=model_uri,
                data=eval_data,
                model_type="question-answering"
            )
            results[version] = result.metrics
    
    # Print comparison
    comparison_df = pd.DataFrame(results).T
    print(comparison_df)
    return comparison_df

# Usage
comparison = compare_rag_versions(
    "models:/rag_chatbot/1",
    "models:/rag_chatbot/2",
    eval_data
)
```

## Best Practices

- **Representative test set**: Cover diverse query types, edge cases.
- **Ground truth quality**: Invest in high-quality gold standard answers.
- **Automated + manual**: Combine metrics with human review.
- **Track over time**: Monitor metrics across model iterations.
- **Segment analysis**: Break down metrics by query category.
- **Cost tracking**: Log token usage, latency alongside quality metrics.

## Sample Questions

1. What does faithfulness measure?
2. How create evaluation dataset?
3. Difference between context precision and recall?
4. When use custom metrics?
5. How compare two RAG models?

## Answers

1. Whether answer is grounded in retrieved context (no hallucinations).
2. Collect representative questions + ground truth answers + relevant sources; use real user queries when possible.
3. Precision: % of retrieved docs that are relevant; Recall: % of relevant docs that were retrieved.
4. When domain-specific quality aspects matter (e.g., citation accuracy, regulatory compliance).
5. Use `mlflow.evaluate()` on same test set for both; compare metrics side-by-side; run A/B test in production.

## References

- [MLflow Model Evaluation](https://mlflow.org/docs/latest/model-evaluation/index.html)
- [RAGAS Framework](https://docs.ragas.io/)
- [Databricks GenAI Evaluation](https://docs.databricks.com/en/generative-ai/agent-evaluation/index.html)

---

Previous: [Masking Techniques as Guardrails](../05-governance/01-masking-techniques-guardrails.md)  
Next: [Inference Logging and Monitoring](./02-inference-logging-monitoring.md)
