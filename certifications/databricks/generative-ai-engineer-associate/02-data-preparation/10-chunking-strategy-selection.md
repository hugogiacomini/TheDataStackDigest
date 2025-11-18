# Selecting Chunking Strategy Based on Model and Retrieval Evaluation

## Overview

Choose optimal chunking approach by evaluating impact on retrieval metrics (precision, recall) and model performance (answer quality, latency).

## Evaluation Framework

```python
class ChunkingEvaluator:
    def __init__(self, test_queries, ground_truth):
        self.test_queries = test_queries
        self.ground_truth = ground_truth
    
    def evaluate_strategy(self, chunking_strategy, embedding_model, vectorstore):
        """Evaluate chunking strategy end-to-end."""
        
        # Re-chunk documents
        chunks = chunking_strategy.chunk_documents(documents)
        
        # Re-embed and index
        vectorstore.add_documents(chunks)
        
        # Evaluate retrieval
        retrieval_metrics = self.compute_retrieval_metrics(vectorstore)
        
        # Evaluate answer quality (with LLM)
        answer_metrics = self.compute_answer_quality(vectorstore)
        
        return {
            **retrieval_metrics,
            **answer_metrics,
            "avg_chunk_size": np.mean([len(c.page_content) for c in chunks]),
            "total_chunks": len(chunks)
        }
    
    def compute_retrieval_metrics(self, vectorstore):
        precision_scores = []
        recall_scores = []
        
        for query, relevant_docs in zip(self.test_queries, self.ground_truth):
            retrieved = vectorstore.similarity_search(query, k=5)
            retrieved_ids = [doc.metadata['id'] for doc in retrieved]
            
            hits = len(set(retrieved_ids) & set(relevant_docs))
            precision_scores.append(hits / 5)
            recall_scores.append(hits / len(relevant_docs))
        
        return {
            "precision@5": np.mean(precision_scores),
            "recall@5": np.mean(recall_scores)
        }
    
    def compute_answer_quality(self, vectorstore):
        """Use LLM judge to score answers."""
        # Implement with RAGAS or custom LLM evaluation
        pass
```

## Strategy Selection Decision Tree

```text
Document Type?
├─ Code / Structured
│  └─ Use: Structure-aware (language-specific)
│     Chunk size: 200-500 tokens
│
├─ Long-form Narrative
│  └─ Use: Semantic or hierarchical
│     Chunk size: 500-1000 tokens
│
├─ Technical Docs with Sections
│  └─ Use: Markdown-header-based
│     Chunk size: Align with sections
│
└─ Mixed Content (tables + text)
   └─ Use: Unstructured with type preservation
      Chunk size: Keep tables intact
```

## Comparative Analysis Example

```python
import pandas as pd

def compare_chunking_strategies(documents, test_queries):
    """Compare multiple strategies."""
    
    strategies = {
        "fixed_500": RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50),
        "fixed_1000": RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=100),
        "semantic": SemanticChunker(embedding_model),
        "hierarchical": HierarchicalChunker(parent_size=2000, child_size=500)
    }
    
    results = []
    for name, strategy in strategies.items():
        metrics = evaluate_strategy(strategy, documents, test_queries)
        metrics['strategy'] = name
        results.append(metrics)
    
    df = pd.DataFrame(results)
    print(df.sort_values('precision@5', ascending=False))
    return df
```

## Selection Criteria

| Factor | Consideration |
|--------|--------------|
| **Embedding Model Context** | Chunk size ≤ 80% of context window |
| **Query Complexity** | Simple queries → smaller chunks; complex → larger |
| **Domain** | Technical docs → structure-aware; narrative → semantic |
| **Latency Budget** | More chunks = slower retrieval; balance with precision |
| **Storage Cost** | Smaller chunks = more embeddings = higher cost |

## Best Practices

- **Baseline first**: Start with fixed-size 500-token chunks, 10% overlap.
- **Measure, don't guess**: Run evaluation on representative test set.
- **Iterate**: Adjust chunk size, overlap, and strategy based on metrics.
- **Consider user feedback**: Combine quantitative metrics with qualitative assessment.
- **Document tradeoffs**: Record why strategy was chosen for reproducibility.

## Sample Questions

1. How determine optimal chunk size?
2. When use hierarchical vs flat chunking?
3. Impact of chunk overlap on retrieval?
4. How evaluate chunking strategy?
5. Red flags in chunking evaluation?

## Answers

1. Test multiple sizes; measure retrieval precision and answer completeness; consider embedding model limits.
2. Hierarchical when need precise retrieval but broad context for LLM; flat for simplicity and speed.
3. Overlap preserves boundary context, improves recall; excessive overlap increases redundancy and cost.
4. Retrieval metrics (precision, recall), answer quality (RAGAS, human eval), latency, cost.
5. High chunk count with low precision (over-chunked), large chunks with poor recall (under-chunked), excessive latency.

## References

- [Chunking Evaluation Guide](https://www.pinecone.io/learn/chunking-strategies/)
- [RAGAS Evaluation](https://docs.ragas.io/)

---

Previous: [Re-Ranking in Information Retrieval](./09-reranking-in-retrieval.md)  
Next: [Qualitative Response Assessment](../03-application-development/01-qualitative-response-assessment.md)
