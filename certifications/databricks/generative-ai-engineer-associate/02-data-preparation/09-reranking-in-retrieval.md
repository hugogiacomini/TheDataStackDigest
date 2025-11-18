# The Role of Re-Ranking in Information Retrieval

## Overview

Explain how re-ranking improves retrieval precision by reordering initial results using more sophisticated models (cross-encoders, LLM-based rerankers).

## Retrieval Pipeline with Re-Ranking

```text
Query → Embedding → Vector Search (top-100) → Re-Ranker → Top-5 → LLM
        (fast, coarse)                          (slow, precise)
```

**Two-Stage Retrieval**:

1. **Stage 1 (Retrieval)**: Fast embedding similarity search → many candidates (e.g., top-100).
2. **Stage 2 (Re-Ranking)**: Precise relevance scoring → final top-K (e.g., top-5).

## Why Re-Rank?

- **Embedding limitations**: Bi-encoders (used in vector search) encode query and document separately; miss query-document interaction.
- **Precision improvement**: Cross-encoders jointly encode query+document for better relevance.
- **Cost optimization**: Expensive models only on top candidates.

## Hands-on Examples

### Cross-Encoder Re-Ranker

```python
from sentence_transformers import CrossEncoder

def rerank_documents(query: str, documents: List[str], top_k=5):
    """Re-rank documents using cross-encoder."""
    
    model = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
    
    # Score each doc with query
    pairs = [[query, doc] for doc in documents]
    scores = model.predict(pairs)
    
    # Sort by score
    ranked_indices = np.argsort(scores)[::-1][:top_k]
    reranked = [(documents[i], scores[i]) for i in ranked_indices]
    
    return reranked

# Example
query = "How to optimize Spark joins?"
candidates = [
    "Broadcast small tables to avoid shuffle in Spark joins",
    "Delta Lake provides ACID transactions",
    "Use Z-ORDER for better file pruning",
    "Join optimization with AQE in Spark 3.0"
]

reranked = rerank_documents(query, candidates, top_k=2)
for doc, score in reranked:
    print(f"Score: {score:.3f} - {doc}")
```

### LLM-Based Re-Ranking

```python
from langchain_community.chat_models import ChatDatabricks

def llm_rerank(query: str, documents: List[str], top_k=5):
    """Use LLM to score relevance."""
    
    llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
    
    scored_docs = []
    for doc in documents:
        prompt = f"""
        Rate the relevance of this document to the query on a scale of 0-10.
        Only respond with a number.
        
        Query: {query}
        Document: {doc}
        
        Relevance score:
        """
        
        score_str = llm.invoke(prompt).content.strip()
        try:
            score = float(score_str)
        except:
            score = 0.0
        
        scored_docs.append((doc, score))
    
    # Sort and return top-k
    scored_docs.sort(key=lambda x: x[1], reverse=True)
    return scored_docs[:top_k]
```

### Cohere Re-Rank API

```python
import cohere

def cohere_rerank(query: str, documents: List[str], top_k=5):
    """Use Cohere re-rank endpoint."""
    
    co = cohere.Client(api_key="YOUR_API_KEY")
    
    results = co.rerank(
        query=query,
        documents=documents,
        top_n=top_k,
        model="rerank-english-v2.0"
    )
    
    return [(doc.document['text'], doc.relevance_score) for doc in results]
```

### Integrated RAG with Re-Ranking

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CohereRerank
from langchain_community.vectorstores import DatabricksVectorSearch

# Base retriever (gets top-100)
base_retriever = vectorstore.as_retriever(search_kwargs={"k": 100})

# Re-ranker
compressor = CohereRerank(top_n=5, model="rerank-english-v2.0")

# Combined retriever
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever
)

# Use in chain
docs = compression_retriever.get_relevant_documents("What are Spark optimizations?")
```

## Re-Ranking Models Comparison

| Model | Speed | Accuracy | Cost | Best For |
|-------|-------|----------|------|----------|
| Cross-Encoder (MiniLM) | Fast | Good | Free (self-host) | General purpose |
| Cohere Rerank | Medium | Excellent | API cost | Production RAG |
| LLM-based (GPT-4) | Slow | Very good | High | Complex relevance |
| BGE-Reranker | Fast | Very good | Free | Open-source RAG |

## Best Practices

- Retrieve **more candidates** in stage 1 (50-100) than needed.
- **Tune top-k** for re-ranker based on latency budget.
- **Cache scores** for repeated queries.
- **Monitor latency**: Re-ranking adds overhead; profile end-to-end.
- **A/B test**: Measure retrieval improvement vs added cost.

## Sample Questions

1. Why re-rank after vector search?
2. Difference between bi-encoder and cross-encoder?
3. When use LLM for re-ranking?
4. Typical retrieval→rerank candidate counts?
5. Trade-off of re-ranking?

## Answers

1. Improves precision; vector search optimizes recall, re-ranker refines top results.
2. Bi-encoder encodes separately (fast, parallel); cross-encoder encodes jointly (slow, accurate).
3. When nuanced understanding needed (legal, medical); cost and latency acceptable.
4. Retrieve 50-100, re-rank to top 5-10.
5. Better precision but added latency and compute cost; worthwhile for high-value queries.

## References

- [Cross-Encoders for Re-Ranking](https://www.sbert.net/examples/applications/cross-encoder/README.html)
- [Cohere Rerank](https://docs.cohere.com/docs/reranking)
- [LangChain Contextual Compression](https://python.langchain.com/docs/modules/data_connection/retrievers/contextual_compression)

---

Previous: [Advanced Chunking Strategies](./08-advanced-chunking-strategies.md)  
Next: [Chunking Strategy Selection Based on Evaluation](./10-chunking-strategy-selection.md)
