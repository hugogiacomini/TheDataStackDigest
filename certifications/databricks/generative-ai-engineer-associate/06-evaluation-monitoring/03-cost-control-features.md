# Using Databricks Features to Control LLM Costs

## Overview

Implement cost control strategies for LLM applications: model selection, caching, batch processing, monitoring, and budget alerts in Databricks.

## Cost Optimization Strategies

| Strategy | Savings Potential | Implementation |
|----------|------------------|----------------|
| **Model rightsizing** | 5-10x | Use smaller models for simple tasks |
| **Semantic caching** | 50-90% | Cache similar query responses |
| **Batch processing** | 30-50% | Use ai_query() vs real-time endpoints |
| **Prompt optimization** | 20-40% | Shorter prompts, efficient formatting |
| **Request throttling** | Variable | Rate limits, user quotas |

## Hands-on Examples

### Model Cost Comparison

```python
import pandas as pd

def compare_model_costs(queries_per_day: int, avg_tokens_per_query: int):
    """Compare costs across model options."""
    
    models = [
        {"name": "Llama 3.1 8B", "cost_per_1k_input": 0.00005, "cost_per_1k_output": 0.0001},
        {"name": "Llama 3.1 70B", "cost_per_1k_input": 0.0005, "cost_per_1k_output": 0.001},
        {"name": "Llama 3.1 405B", "cost_per_1k_input": 0.0025, "cost_per_1k_output": 0.005},
    ]
    
    results = []
    
    # Assume 30% input, 70% output tokens
    input_tokens = avg_tokens_per_query * 0.3
    output_tokens = avg_tokens_per_query * 0.7
    
    for model in models:
        daily_input_cost = (queries_per_day * input_tokens / 1000) * model["cost_per_1k_input"]
        daily_output_cost = (queries_per_day * output_tokens / 1000) * model["cost_per_1k_output"]
        daily_total = daily_input_cost + daily_output_cost
        
        results.append({
            "Model": model["name"],
            "Daily Cost": f"${daily_total:.2f}",
            "Monthly Cost": f"${daily_total * 30:.2f}",
            "Annual Cost": f"${daily_total * 365:.2f}"
        })
    
    return pd.DataFrame(results)

# Example: 10K queries/day, 500 tokens average
cost_comparison = compare_model_costs(10000, 500)
print(cost_comparison)
```

### Semantic Cache Implementation

```python
from langchain.embeddings import DatabricksEmbeddings
from langchain.vectorstores import FAISS
import hashlib

class SemanticCache:
    def __init__(self, similarity_threshold: float = 0.95):
        self.embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")
        self.cache_store = {}  # In production: use Redis or Delta
        self.query_embeddings = []
        self.threshold = similarity_threshold
    
    def get(self, query: str):
        """Check if similar query exists in cache."""
        
        # Compute query embedding
        query_embedding = self.embeddings.embed_query(query)
        
        # Check similarity with cached queries
        for cached_query, cached_embedding, cached_response in self.cache_store.values():
            similarity = self._cosine_similarity(query_embedding, cached_embedding)
            
            if similarity >= self.threshold:
                print(f"Cache HIT (similarity: {similarity:.3f})")
                return cached_response
        
        print("Cache MISS")
        return None
    
    def set(self, query: str, response: str):
        """Cache query-response pair."""
        
        query_embedding = self.embeddings.embed_query(query)
        cache_key = hashlib.md5(query.encode()).hexdigest()
        
        self.cache_store[cache_key] = (query, query_embedding, response)
    
    def _cosine_similarity(self, vec1, vec2):
        import numpy as np
        return np.dot(vec1, vec2) / (np.linalg.norm(vec1) * np.linalg.norm(vec2))

# Usage with RAG
cache = SemanticCache(similarity_threshold=0.95)

def cached_rag_query(query: str) -> str:
    """RAG with semantic caching."""
    
    # Check cache
    cached_response = cache.get(query)
    if cached_response:
        return cached_response
    
    # Generate response (costs tokens)
    response = expensive_llm_call(query)
    
    # Cache for future
    cache.set(query, response)
    
    return response

# Test
print(cached_rag_query("What is Unity Catalog?"))  # MISS - calls LLM
print(cached_rag_query("What's Unity Catalog?"))  # HIT - uses cache (similar query)
```

### Token Usage Tracking

```python
from datetime import datetime
import pandas as pd

class TokenUsageTracker:
    def __init__(self):
        self.usage_log = []
    
    def log_request(self, user_id: str, model: str, input_tokens: int, 
                   output_tokens: int, cost: float):
        """Log token usage for cost tracking."""
        
        self.usage_log.append({
            "timestamp": datetime.now(),
            "user_id": user_id,
            "model": model,
            "input_tokens": input_tokens,
            "output_tokens": output_tokens,
            "total_tokens": input_tokens + output_tokens,
            "cost": cost
        })
    
    def get_daily_cost(self, date: str = None) -> float:
        """Get total cost for a day."""
        
        if date is None:
            date = datetime.now().date()
        
        df = pd.DataFrame(self.usage_log)
        daily = df[df['timestamp'].dt.date == pd.to_datetime(date).date()]
        
        return daily['cost'].sum()
    
    def get_user_usage(self, user_id: str) -> dict:
        """Get usage stats for user."""
        
        df = pd.DataFrame(self.usage_log)
        user_df = df[df['user_id'] == user_id]
        
        return {
            "total_requests": len(user_df),
            "total_tokens": user_df['total_tokens'].sum(),
            "total_cost": user_df['cost'].sum(),
            "avg_tokens_per_request": user_df['total_tokens'].mean()
        }

tracker = TokenUsageTracker()

# Log usage after each request
def tracked_llm_call(user_id: str, query: str) -> str:
    response = llm.invoke(query)
    
    # Extract token counts (from response metadata)
    input_tokens = len(query.split()) * 1.3  # Rough estimate
    output_tokens = len(response.content.split()) * 1.3
    
    # Calculate cost
    cost_per_1k = 0.001  # Llama 70B
    cost = ((input_tokens + output_tokens) / 1000) * cost_per_1k
    
    tracker.log_request(user_id, "llama-70b", int(input_tokens), int(output_tokens), cost)
    
    return response.content
```

### Budget Alerts

```python
class BudgetMonitor:
    def __init__(self, daily_budget: float, monthly_budget: float):
        self.daily_budget = daily_budget
        self.monthly_budget = monthly_budget
        self.tracker = TokenUsageTracker()
    
    def check_budget(self) -> dict:
        """Check if spending is within budget."""
        
        today_cost = self.tracker.get_daily_cost()
        month_cost = self._get_monthly_cost()
        
        alerts = []
        
        # Daily budget check
        if today_cost >= self.daily_budget:
            alerts.append(f"DAILY BUDGET EXCEEDED: ${today_cost:.2f} / ${self.daily_budget:.2f}")
        elif today_cost >= self.daily_budget * 0.8:
            alerts.append(f"DAILY BUDGET WARNING: ${today_cost:.2f} / ${self.daily_budget:.2f} (80%)")
        
        # Monthly budget check
        if month_cost >= self.monthly_budget:
            alerts.append(f"MONTHLY BUDGET EXCEEDED: ${month_cost:.2f} / ${self.monthly_budget:.2f}")
        elif month_cost >= self.monthly_budget * 0.8:
            alerts.append(f"MONTHLY BUDGET WARNING: ${month_cost:.2f} / ${self.monthly_budget:.2f} (80%)")
        
        return {
            "within_budget": len(alerts) == 0,
            "daily_cost": today_cost,
            "monthly_cost": month_cost,
            "alerts": alerts
        }
    
    def _get_monthly_cost(self) -> float:
        df = pd.DataFrame(self.tracker.usage_log)
        current_month = datetime.now().month
        monthly = df[df['timestamp'].dt.month == current_month]
        return monthly['cost'].sum()

budget_monitor = BudgetMonitor(daily_budget=100, monthly_budget=2500)
```

### Prompt Optimization for Cost

```python
def optimize_prompt_length(query: str, context: str, max_tokens: int = 2000) -> str:
    """Trim prompt to reduce cost while preserving info."""
    
    # Estimate tokens (rough: 1 token ≈ 4 chars)
    def estimate_tokens(text: str) -> int:
        return len(text) // 4
    
    # Build prompt
    base_prompt = f"Question: {query}\n\nAnswer:"
    base_tokens = estimate_tokens(base_prompt)
    
    # Calculate available tokens for context
    available_tokens = max_tokens - base_tokens - 200  # Reserve for response
    available_chars = available_tokens * 4
    
    # Truncate context if needed
    if len(context) > available_chars:
        context = context[:available_chars] + "..."
        print(f"Truncated context to fit budget ({available_tokens} tokens)")
    
    return f"{base_prompt}\n\nContext: {context}\n\nAnswer:"

# Usage
long_context = "..." * 10000  # Very long
optimized_prompt = optimize_prompt_length("What is Delta?", long_context, max_tokens=2000)
```

### Batch vs Real-Time Cost Analysis

```python
def compare_batch_vs_realtime(num_queries: int, avg_tokens: int):
    """Compare costs: batch (ai_query) vs real-time (endpoint)."""
    
    # Batch processing (ai_query)
    batch_cost_per_1k = 0.001
    batch_cost = (num_queries * avg_tokens / 1000) * batch_cost_per_1k
    batch_time = num_queries / 50  # Assume 50 queries/sec throughput
    
    # Real-time endpoint
    realtime_cost_per_1k = 0.001  # Same model
    realtime_infrastructure_cost = 200  # Monthly endpoint cost
    realtime_cost = (num_queries * avg_tokens / 1000) * realtime_cost_per_1k + realtime_infrastructure_cost
    realtime_time = num_queries / 10  # Assume 10 queries/sec for real-time
    
    return pd.DataFrame([
        {
            "Method": "Batch (ai_query)",
            "Token Cost": f"${batch_cost:.2f}",
            "Infrastructure": "$0",
            "Total Cost": f"${batch_cost:.2f}",
            "Time": f"{batch_time:.1f}s"
        },
        {
            "Method": "Real-Time Endpoint",
            "Token Cost": f"${batch_cost:.2f}",
            "Infrastructure": f"${realtime_infrastructure_cost:.2f}",
            "Total Cost": f"${realtime_cost:.2f}",
            "Time": f"{realtime_time:.1f}s"
        }
    ])

comparison = compare_batch_vs_realtime(10000, 500)
print(comparison)
```

## Best Practices

- **Right-size models**: Use 8B for simple tasks; save 70B/405B for complex.
- **Implement caching**: Semantic cache for similar queries; exact match cache.
- **Monitor actively**: Track costs daily; set alerts at 80% of budget.
- **Batch when possible**: Use ai_query() for non-time-sensitive workloads.
- **Optimize prompts**: Shorter prompts, efficient formatting, truncate context.
- **User quotas**: Implement per-user limits to prevent runaway costs.

## Sample Questions

1. How reduce LLM costs by 10x?
2. What is semantic caching?
3. When use batch vs real-time?
4. How track token usage?
5. Best practice for budget alerts?

## Answers

1. Use 8B model instead of 405B for simple tasks (10x cheaper per token); add semantic caching (50-90% cache hit rate).
2. Cache responses for semantically similar queries (e.g., "What is X?" and "What's X?" treated as same); use embedding similarity.
3. Batch (ai_query) for large datasets, non-urgent (classification, summarization); real-time (endpoint) for user-facing, low-latency needs.
4. Log input/output tokens per request; multiply by model cost per 1K tokens; store in Delta table; aggregate daily/monthly.
5. Set alerts at 80% of daily/monthly budget; send to Slack/email; implement hard limits at 100%; review usage weekly.

## References

- [Databricks Model Pricing](https://www.databricks.com/product/pricing/model-serving)
- [Cost Optimization Guide](https://docs.databricks.com/en/machine-learning/model-serving/cost-optimization.html)

---

Previous: [Inference Logging and Monitoring](./02-inference-logging-monitoring.md)  
Next: [Evaluation Judges Requiring Ground Truth](./04-evaluation-judges-ground-truth.md)
