# Model Selection Based on Application Requirements

## Overview

Select appropriate LLMs and embedding models based on application constraints: latency, accuracy, cost, context length, and domain specificity.

## Selection Criteria

| Factor | Considerations | Tools |
|--------|---------------|-------|
| **Latency** | Response time requirements | Smaller models (7B-13B), optimized serving |
| **Accuracy** | Task complexity | Larger models (70B+), fine-tuned |
| **Cost** | Token pricing, volume | Open-source vs API, caching |
| **Context Length** | Input size needs | 4K, 8K, 32K, 128K+ context windows |
| **Domain** | Specialized knowledge | Domain-specific vs general models |

## Hands-on Examples

### LLM Selection Framework

```python
from dataclasses import dataclass
from typing import List

@dataclass
class ModelRequirements:
    max_latency_ms: int
    min_accuracy: float
    max_cost_per_1k_tokens: float
    min_context_length: int
    domain: str
    tasks: List[str]

@dataclass
class ModelCandidate:
    name: str
    latency_p95_ms: int
    accuracy_score: float
    cost_per_1k_tokens: float
    context_length: int
    domains: List[str]
    endpoint: str

# Available models
AVAILABLE_MODELS = [
    ModelCandidate(
        name="Llama 3.1 8B Instruct",
        latency_p95_ms=150,
        accuracy_score=0.75,
        cost_per_1k_tokens=0.0001,
        context_length=128000,
        domains=["general"],
        endpoint="databricks-meta-llama-3-1-8b-instruct"
    ),
    ModelCandidate(
        name="Llama 3.1 70B Instruct",
        latency_p95_ms=800,
        accuracy_score=0.90,
        cost_per_1k_tokens=0.001,
        context_length=128000,
        domains=["general", "code", "reasoning"],
        endpoint="databricks-meta-llama-3-1-70b-instruct"
    ),
    ModelCandidate(
        name="Llama 3.1 405B Instruct",
        latency_p95_ms=3000,
        accuracy_score=0.95,
        cost_per_1k_tokens=0.005,
        context_length=128000,
        domains=["general", "code", "reasoning", "complex"],
        endpoint="databricks-meta-llama-3-1-405b-instruct"
    ),
    ModelCandidate(
        name="DBRX Instruct",
        latency_p95_ms=600,
        accuracy_score=0.88,
        cost_per_1k_tokens=0.0008,
        context_length=32000,
        domains=["general", "code"],
        endpoint="databricks-dbrx-instruct"
    )
]

def select_llm(requirements: ModelRequirements) -> List[ModelCandidate]:
    """Select suitable LLM based on requirements."""
    
    candidates = []
    
    for model in AVAILABLE_MODELS:
        # Check hard constraints
        if model.latency_p95_ms > requirements.max_latency_ms:
            continue
        if model.accuracy_score < requirements.min_accuracy:
            continue
        if model.cost_per_1k_tokens > requirements.max_cost_per_1k_tokens:
            continue
        if model.context_length < requirements.min_context_length:
            continue
        
        # Check domain match
        if requirements.domain != "general" and requirements.domain not in model.domains:
            continue
        
        candidates.append(model)
    
    # Sort by accuracy (descending), then cost (ascending)
    candidates.sort(key=lambda m: (-m.accuracy_score, m.cost_per_1k_tokens))
    
    return candidates

# Example usage
reqs = ModelRequirements(
    max_latency_ms=1000,
    min_accuracy=0.85,
    max_cost_per_1k_tokens=0.002,
    min_context_length=32000,
    domain="general",
    tasks=["question_answering", "summarization"]
)

recommended_models = select_llm(reqs)
for model in recommended_models:
    print(f"{model.name}: {model.accuracy_score} accuracy, ${model.cost_per_1k_tokens}/1K tokens, {model.latency_p95_ms}ms")
```

### Embedding Model Selection

```python
@dataclass
class EmbeddingModelCandidate:
    name: str
    dimension: int
    max_tokens: int
    latency_ms: int
    quality_score: float
    endpoint: str

EMBEDDING_MODELS = [
    EmbeddingModelCandidate(
        name="BGE Large EN",
        dimension=1024,
        max_tokens=512,
        latency_ms=50,
        quality_score=0.90,
        endpoint="databricks-bge-large-en"
    ),
    EmbeddingModelCandidate(
        name="GTE Large",
        dimension=1024,
        max_tokens=512,
        latency_ms=45,
        quality_score=0.88,
        endpoint="databricks-gte-large-en"
    ),
    EmbeddingModelCandidate(
        name="OpenAI text-embedding-3-large",
        dimension=3072,
        max_tokens=8191,
        latency_ms=100,
        quality_score=0.92,
        endpoint="openai-text-embedding-3-large"
    )
]

def select_embedding_model(
    avg_chunk_size: int,
    retrieval_precision_required: float,
    max_latency_ms: int
) -> EmbeddingModelCandidate:
    """Select embedding model based on requirements."""
    
    candidates = []
    
    for model in EMBEDDING_MODELS:
        # Check if model can handle chunk size
        if avg_chunk_size > model.max_tokens:
            continue
        
        # Check latency
        if model.latency_ms > max_latency_ms:
            continue
        
        # Check quality
        if model.quality_score < retrieval_precision_required:
            continue
        
        candidates.append(model)
    
    # Prefer higher quality, then lower latency
    candidates.sort(key=lambda m: (-m.quality_score, m.latency_ms))
    
    return candidates[0] if candidates else None

# Usage
embedding_model = select_embedding_model(
    avg_chunk_size=400,
    retrieval_precision_required=0.85,
    max_latency_ms=100
)
print(f"Selected: {embedding_model.name}")
```

### Cost-Performance Trade-off Analysis

```python
import pandas as pd

def analyze_cost_performance(workload_queries_per_day: int, avg_tokens_per_query: int):
    """Compare models on cost-performance."""
    
    models = [
        {"name": "Llama 3.1 8B", "accuracy": 0.75, "latency": 150, "cost_per_1k": 0.0001},
        {"name": "Llama 3.1 70B", "accuracy": 0.90, "latency": 800, "cost_per_1k": 0.001},
        {"name": "Llama 3.1 405B", "accuracy": 0.95, "latency": 3000, "cost_per_1k": 0.005}
    ]
    
    results = []
    for model in models:
        daily_cost = (workload_queries_per_day * avg_tokens_per_query / 1000) * model["cost_per_1k"]
        monthly_cost = daily_cost * 30
        
        results.append({
            "Model": model["name"],
            "Accuracy": model["accuracy"],
            "P95 Latency (ms)": model["latency"],
            "Daily Cost ($)": round(daily_cost, 2),
            "Monthly Cost ($)": round(monthly_cost, 2),
            "Cost per 0.01 Accuracy": round(monthly_cost / model["accuracy"] * 0.01, 2)
        })
    
    return pd.DataFrame(results)

# Example: 10K queries/day, 500 tokens/query average
comparison = analyze_cost_performance(10000, 500)
print(comparison)
```

### Task-Specific Model Recommendations

```python
TASK_MODEL_MAP = {
    "simple_qa": {
        "recommended": "databricks-meta-llama-3-1-8b-instruct",
        "reason": "Fast, sufficient for straightforward questions"
    },
    "complex_reasoning": {
        "recommended": "databricks-meta-llama-3-1-70b-instruct",
        "reason": "Better chain-of-thought, multi-step reasoning"
    },
    "code_generation": {
        "recommended": "databricks-dbrx-instruct",
        "reason": "Optimized for code, strong on Python/SQL"
    },
    "long_document_qa": {
        "recommended": "databricks-meta-llama-3-1-70b-instruct",
        "reason": "128K context, good comprehension"
    },
    "summarization": {
        "recommended": "databricks-meta-llama-3-1-8b-instruct",
        "reason": "Fast, cost-effective for extraction tasks"
    },
    "creative_writing": {
        "recommended": "databricks-meta-llama-3-1-405b-instruct",
        "reason": "Best quality for nuanced, creative outputs"
    }
}

def recommend_model_for_task(task: str, constraints: dict = None) -> str:
    """Get model recommendation for task."""
    
    if task not in TASK_MODEL_MAP:
        return "databricks-meta-llama-3-1-70b-instruct"  # Default
    
    recommendation = TASK_MODEL_MAP[task]
    
    # Override if latency constraint
    if constraints and constraints.get("max_latency_ms", float('inf')) < 500:
        return "databricks-meta-llama-3-1-8b-instruct"
    
    return recommendation["recommended"]
```

### A/B Testing Model Selection

```python
class ModelSelector:
    def __init__(self):
        self.model_stats = {}
    
    def select_for_request(self, request_id: str, user_id: str = None) -> str:
        """Select model using A/B test (traffic split)."""
        
        # Simple hash-based routing (consistent per user)
        if user_id:
            hash_val = hash(user_id) % 100
        else:
            hash_val = hash(request_id) % 100
        
        # 70% traffic to Llama 70B, 30% to DBRX
        if hash_val < 70:
            return "databricks-meta-llama-3-1-70b-instruct"
        else:
            return "databricks-dbrx-instruct"
    
    def log_performance(self, model: str, latency: float, user_rating: int):
        """Track model performance."""
        if model not in self.model_stats:
            self.model_stats[model] = {"latencies": [], "ratings": []}
        
        self.model_stats[model]["latencies"].append(latency)
        self.model_stats[model]["ratings"].append(user_rating)
    
    def get_winner(self) -> str:
        """Determine better performing model."""
        results = {}
        for model, stats in self.model_stats.items():
            avg_latency = sum(stats["latencies"]) / len(stats["latencies"])
            avg_rating = sum(stats["ratings"]) / len(stats["ratings"])
            
            # Combined score (higher rating better, lower latency better)
            score = avg_rating - (avg_latency / 1000)  # Normalize latency
            results[model] = score
        
        return max(results, key=results.get)
```

## Best Practices

- **Start conservative**: Begin with larger model; downgrade if latency/cost too high.
- **Measure, don't assume**: Run benchmarks on your specific use case.
- **Task segmentation**: Route simple queries to fast models, complex to large models.
- **Context length padding**: Choose model with 2x your typical context for safety.
- **Cost monitoring**: Track token usage; adjust if exceeds budget.
- **A/B test**: Compare models in production with real traffic.

## Sample Questions

1. How choose between 8B and 70B model?
2. Key factor for embedding model selection?
3. When use largest (405B) model?
4. How reduce LLM costs?
5. What is context length headroom?

## Answers

1. If latency <500ms required and task is simple (QA, summarization): 8B. If accuracy critical or complex reasoning: 70B.
2. Max tokens (must handle chunk size), retrieval quality score (precision@K on eval set), latency budget.
3. For highest accuracy needs (creative writing, complex reasoning), when accuracy worth 5-10x cost, or when smaller models fail eval.
4. Use smaller models for simple tasks, cache responses, batch requests, use open-source models, implement semantic cache.
5. Choose model context length 2x your typical input; leaves room for retrieval expansion, prevents truncation errors.

## References

- [Databricks Foundation Model APIs](https://docs.databricks.com/en/machine-learning/foundation-models/index.html)
- [Model Comparison](https://artificialanalysis.ai/)

---

Previous: [Writing Metaprompts](./05-writing-metaprompts-anti-hallucination.md)  
Next: [Agent Framework Utilization](./07-agent-framework-utilization.md)
