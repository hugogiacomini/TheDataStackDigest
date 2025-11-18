# Retrieval Performance Evaluation: Tools and Metrics

## Overview

Use specialized tools and metrics to evaluate retrieval quality in RAG systems: precision, recall, MRR, NDCG, and retrieval frameworks.

## Key Metrics

| Metric | Definition | Best For |
|--------|-----------|----------|
| **Precision@K** | % of top-K results that are relevant | Measuring quality of top results |
| **Recall@K** | % of all relevant docs found in top-K | Coverage assessment |
| **MRR (Mean Reciprocal Rank)** | Average of 1/rank of first relevant result | Single correct answer tasks |
| **NDCG@K** | Discounted cumulative gain (accounts for position) | Ranked relevance |
| **Hit Rate** | % of queries with ≥1 relevant result in top-K | Minimum success rate |

## Hands-on Examples

### Manual Evaluation Setup

```python
import numpy as np

def precision_at_k(retrieved, relevant, k):
    """Calculate Precision@K."""
    retrieved_k = retrieved[:k]
    hits = len(set(retrieved_k) & set(relevant))
    return hits / k if k > 0 else 0

def recall_at_k(retrieved, relevant, k):
    """Calculate Recall@K."""
    retrieved_k = retrieved[:k]
    hits = len(set(retrieved_k) & set(relevant))
    return hits / len(relevant) if len(relevant) > 0 else 0

def mrr(retrieved_list, relevant_list):
    """Calculate Mean Reciprocal Rank."""
    reciprocal_ranks = []
    for retrieved, relevant in zip(retrieved_list, relevant_list):
        for rank, doc_id in enumerate(retrieved, start=1):
            if doc_id in relevant:
                reciprocal_ranks.append(1 / rank)
                break
        else:
            reciprocal_ranks.append(0)
    return np.mean(reciprocal_ranks)

# Example
retrieved = ["doc1", "doc3", "doc5", "doc2"]
relevant = ["doc2", "doc5"]

print(f"Precision@2: {precision_at_k(retrieved, relevant, 2)}")  # 0.5
print(f"Recall@2: {recall_at_k(retrieved, relevant, 2)}")       # 0.5
print(f"MRR: {mrr([retrieved], [relevant])}")                   # 0.5 (first relevant at rank 2)
```

### Using RAGAS Framework

```python
from ragas import evaluate
from ragas.metrics import (
    context_precision,
    context_recall,
    faithfulness,
    answer_relevancy
)
from datasets import Dataset

# Prepare evaluation dataset
data = {
    'question': ['What is Delta Lake?', 'How does Spark handle partitions?'],
    'answer': ['Delta Lake is an open-source storage layer...', 'Spark distributes data across partitions...'],
    'contexts': [
        ['Delta Lake provides ACID transactions for data lakes...'],
        ['Partitions are the basic unit of parallelism in Spark...']
    ],
    'ground_truth': ['Delta Lake is a storage layer providing ACID transactions', 'Spark uses partitions for parallel processing']
}

dataset = Dataset.from_dict(data)

# Evaluate
result = evaluate(
    dataset,
    metrics=[context_precision, context_recall, faithfulness, answer_relevancy]
)

print(result)
```

### Databricks MLflow Evaluation

```python
import mlflow
import pandas as pd

# Define evaluation dataset
eval_data = pd.DataFrame({
    "inputs": ["What are the benefits of Unity Catalog?"],
    "ground_truth": ["Centralized governance, fine-grained access control, data lineage"]
})

# Evaluate RAG model
with mlflow.start_run():
    results = mlflow.evaluate(
        model="models:/rag_model/production",
        data=eval_data,
        model_type="question-answering",
        evaluators="default"
    )
    
    print(results.metrics)
```

## Retrieval Evaluation Pipeline

```python
from typing import List, Dict
from langchain_community.vectorstores import DatabricksVectorSearch
from langchain_community.embeddings import DatabricksEmbeddings

class RetrievalEvaluator:
    def __init__(self, vectorstore, test_queries: List[Dict]):
        self.vectorstore = vectorstore
        self.test_queries = test_queries  # [{"query": "...", "relevant_docs": [...]}]
    
    def evaluate(self, k=5):
        metrics = {
            "precision@k": [],
            "recall@k": [],
            "mrr": []
        }
        
        for test in self.test_queries:
            query = test["query"]
            relevant = set(test["relevant_docs"])
            
            # Retrieve
            results = self.vectorstore.similarity_search(query, k=k)
            retrieved = [doc.metadata.get("chunk_id") for doc in results]
            
            # Calculate metrics
            hits = len(set(retrieved) & relevant)
            metrics["precision@k"].append(hits / k)
            metrics["recall@k"].append(hits / len(relevant) if len(relevant) > 0 else 0)
            
            # MRR
            for rank, doc_id in enumerate(retrieved, start=1):
                if doc_id in relevant:
                    metrics["mrr"].append(1 / rank)
                    break
            else:
                metrics["mrr"].append(0)
        
        # Aggregate
        return {
            "avg_precision@k": np.mean(metrics["precision@k"]),
            "avg_recall@k": np.mean(metrics["recall@k"]),
            "mrr": np.mean(metrics["mrr"])
        }

# Usage
# evaluator = RetrievalEvaluator(vectorstore, test_queries)
# results = evaluator.evaluate(k=5)
```

## Best Practices

- Create **gold standard** test set with human-annotated relevance judgments.
- Evaluate across **diverse query types** (simple, complex, ambiguous).
- Track metrics over time to detect **retrieval drift**.
- Use **A/B testing** for production improvements (chunking, embedding models).
- Combine **quantitative metrics** with **qualitative review** of failure cases.

## Sample Questions

1. What does Precision@3 measure?
2. When use MRR vs NDCG?
3. How create evaluation dataset?
4. What is a good Recall@5 score?
5. Tools for automated RAG evaluation?

## Answers

1. Percentage of top 3 retrieved docs that are relevant.
2. MRR for single relevant answer (QA); NDCG when multiple docs have graded relevance.
3. Collect representative queries + annotate relevant documents manually or via user feedback.
4. Depends on domain; generally >0.7 is good, >0.9 is excellent.
5. RAGAS, MLflow, TruLens, LangSmith, custom eval scripts.

## References

- [RAGAS Framework](https://docs.ragas.io/)
- [MLflow Model Evaluation](https://mlflow.org/docs/latest/model-evaluation/index.html)
- [Information Retrieval Metrics](https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval))

---

Previous: [Prompt/Response Pairs](./06-prompt-response-pairs.md)  
Next: [Advanced Chunking Strategies](./08-advanced-chunking-strategies.md)
