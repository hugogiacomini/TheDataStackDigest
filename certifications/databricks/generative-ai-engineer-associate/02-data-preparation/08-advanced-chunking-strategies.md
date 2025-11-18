# Advanced Chunking Strategies for Retrieval Systems

## Overview

Design sophisticated chunking approaches: hierarchical, sliding window with context, semantic boundary detection, and hybrid strategies.

## Advanced Techniques

### 1. Hierarchical Chunking

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

def hierarchical_chunk(document: str):
    """Create parent-child chunk hierarchy."""
    
    # Parent chunks (larger context)
    parent_splitter = RecursiveCharacterTextSplitter(
        chunk_size=2000,
        chunk_overlap=200
    )
    parent_chunks = parent_splitter.split_text(document)
    
    # Child chunks (retrieval granularity)
    child_splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,
        chunk_overlap=50
    )
    
    hierarchy = []
    for parent_id, parent in enumerate(parent_chunks):
        child_chunks = child_splitter.split_text(parent)
        for child_id, child in enumerate(child_chunks):
            hierarchy.append({
                "parent_id": parent_id,
                "child_id": child_id,
                "parent_text": parent,
                "child_text": child
            })
    
    return hierarchy

# At retrieval: search child chunks, return parent for LLM context
```

### 2. Semantic Boundary Detection

```python
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

def semantic_chunk(sentences: List[str], threshold=0.5):
    """Split at semantic boundaries using embedding similarity."""
    
    model = SentenceTransformer('all-MiniLM-L6-v2')
    embeddings = model.encode(sentences)
    
    chunks = []
    current_chunk = [sentences[0]]
    
    for i in range(1, len(sentences)):
        similarity = cosine_similarity(
            embeddings[i-1].reshape(1, -1),
            embeddings[i].reshape(1, -1)
        )[0][0]
        
        if similarity < threshold:  # Topic shift detected
            chunks.append(' '.join(current_chunk))
            current_chunk = [sentences[i]]
        else:
            current_chunk.append(sentences[i])
    
    chunks.append(' '.join(current_chunk))
    return chunks
```

### 3. Contextual Chunk Enrichment

```python
def enrich_chunks_with_context(chunks, source_metadata):
    """Add surrounding context to chunks."""
    
    enriched = []
    for i, chunk in enumerate(chunks):
        context_before = chunks[i-1] if i > 0 else ""
        context_after = chunks[i+1] if i < len(chunks)-1 else ""
        
        enriched.append({
            "content": chunk,
            "context_before": context_before[-100:],  # Last 100 chars
            "context_after": context_after[:100],      # First 100 chars
            "position": i,
            "total_chunks": len(chunks),
            **source_metadata
        })
    
    return enriched
```

### 4. Proposition-Based Chunking

```python
# Each chunk = single atomic claim/fact
def proposition_chunk(document: str, llm):
    """Extract atomic propositions using LLM."""
    
    prompt = f"""
    Extract atomic propositions from this text. Each proposition should be a single, self-contained fact.
    
    Text: {document}
    
    Output each proposition on a new line.
    """
    
    response = llm.generate(prompt)
    propositions = [p.strip() for p in response.split('\n') if p.strip()]
    return propositions
```

## Hybrid Strategy Example

```python
class HybridChunker:
    def __init__(self, primary_size=500, semantic_model=None):
        self.primary_splitter = RecursiveCharacterTextSplitter(
            chunk_size=primary_size,
            chunk_overlap=50
        )
        self.semantic_model = semantic_model
    
    def chunk(self, document: str, use_semantic=True):
        """Combine fixed-size and semantic chunking."""
        
        # First pass: structural split
        primary_chunks = self.primary_splitter.split_text(document)
        
        if not use_semantic or not self.semantic_model:
            return primary_chunks
        
        # Second pass: refine boundaries
        refined_chunks = []
        for chunk in primary_chunks:
            sentences = chunk.split('. ')
            semantic_chunks = semantic_chunk(sentences)
            refined_chunks.extend(semantic_chunks)
        
        return refined_chunks
```

## Best Practices

- **Test multiple strategies** on your domain; no universal optimum.
- **Combine approaches**: Structure + semantics often outperforms either alone.
- **Preserve metadata**: Track chunk relationships for context reconstruction.
- **Balance granularity**: Smaller = precise retrieval, larger = better context.
- **Monitor impact**: Measure retrieval metrics before/after strategy changes.

## Sample Questions

1. Benefits of hierarchical chunking?
2. When use semantic boundary detection?
3. How enrich chunks with context?
4. Trade-off of proposition-based chunking?
5. Which strategy for legal documents?

## Answers

1. Retrieve fine-grained, provide broad context to LLM; reduces redundancy.
2. When documents have clear topic shifts not aligned with structural markers.
3. Store snippets from adjacent chunks; include source metadata (section, page).
4. Very precise, self-contained facts; more chunks to manage, potential context loss.
5. Hierarchical (clauses = children, sections = parents) + structure-aware to preserve numbering.

## References

- [Advanced RAG Techniques](https://arxiv.org/abs/2312.10997)
- [Semantic Chunking](https://python.langchain.com/docs/modules/data_connection/document_transformers/semantic-chunker)

---

Previous: [Retrieval Performance Evaluation](./07-retrieval-performance-evaluation.md)  
Next: [Re-Ranking in Information Retrieval](./09-reranking-in-retrieval.md)
