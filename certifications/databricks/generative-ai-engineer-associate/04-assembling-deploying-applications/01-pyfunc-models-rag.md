# Coding RAG Chains with PyFunc Models (Pre/Post-Processing)

## Overview

Create custom MLflow pyfunc models with integrated pre-processing (query rewriting, context retrieval) and post-processing (citation extraction, formatting) for production RAG applications.

## Pyfunc Model Structure

```python
import mlflow
from mlflow.pyfunc import PythonModel
from typing import List, Dict
import pandas as pd

class RAGPyfuncModel(PythonModel):
    def __init__(self):
        self.retriever = None
        self.llm = None
    
    def load_context(self, context):
        """Load model artifacts and dependencies."""
        # Initialize retriever, LLM, embeddings
        from langchain_community.vectorstores import DatabricksVectorSearch
        from langchain_community.chat_models import ChatDatabricks
        
        self.retriever = DatabricksVectorSearch(...)
        self.llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
    
    def preprocess(self, query: str) -> Dict:
        """Query enhancement and retrieval."""
        # 1. Query rewriting
        enhanced_query = self._rewrite_query(query)
        
        # 2. Retrieve context
        docs = self.retriever.similarity_search(enhanced_query, k=5)
        context = "\n\n".join([doc.page_content for doc in docs])
        
        return {
            "original_query": query,
            "enhanced_query": enhanced_query,
            "context": context,
            "sources": [doc.metadata.get('source') for doc in docs]
        }
    
    def postprocess(self, llm_response: str, preprocessing_data: Dict) -> Dict:
        """Extract citations, format response."""
        # 1. Extract citations
        citations = self._extract_citations(llm_response, preprocessing_data['sources'])
        
        # 2. Format response
        formatted = {
            "answer": llm_response,
            "citations": citations,
            "sources": preprocessing_data['sources']
        }
        
        return formatted
    
    def predict(self, context, model_input):
        """Main prediction method."""
        if isinstance(model_input, pd.DataFrame):
            queries = model_input['query'].tolist()
        else:
            queries = [model_input]
        
        results = []
        for query in queries:
            # Preprocess
            prep_data = self.preprocess(query)
            
            # Generate
            prompt = f"""Answer the question using only the provided context.

Context: {prep_data['context']}

Question: {prep_data['enhanced_query']}

Answer:"""
            
            llm_response = self.llm.invoke(prompt).content
            
            # Postprocess
            final_output = self.postprocess(llm_response, prep_data)
            results.append(final_output)
        
        return results
    
    def _rewrite_query(self, query: str) -> str:
        """Enhance query for better retrieval."""
        # Simple expansion (production would use LLM)
        return query
    
    def _extract_citations(self, response: str, sources: List[str]) -> List[str]:
        """Match response claims to sources."""
        # Simplified: return all sources
        return sources
```

## Logging and Registration

```python
import mlflow
from mlflow.models.signature import infer_signature

# Example input/output for signature
input_example = pd.DataFrame({"query": ["What is Unity Catalog?"]})
output_example = [{
    "answer": "Unity Catalog is a unified governance solution...",
    "citations": ["doc1.pdf", "doc2.pdf"],
    "sources": ["doc1.pdf", "doc2.pdf"]
}]

# Log model
with mlflow.start_run():
    mlflow.pyfunc.log_model(
        artifact_path="rag_model",
        python_model=RAGPyfuncModel(),
        signature=infer_signature(input_example, output_example),
        input_example=input_example,
        pip_requirements=[
            "langchain==0.1.0",
            "databricks-vectorsearch",
            "databricks-sdk"
        ]
    )
    
    run_id = mlflow.active_run().info.run_id

# Register to Unity Catalog
model_name = "main.ml_models.rag_chatbot"
mlflow.register_model(f"runs:/{run_id}/rag_model", model_name)
```

## Advanced: Chain-of-Thought Preprocessing

```python
class AdvancedRAGPyfunc(PythonModel):
    def preprocess(self, query: str) -> Dict:
        """Multi-stage preprocessing."""
        
        # Stage 1: Query understanding
        intent = self._classify_intent(query)
        
        # Stage 2: Query decomposition (for complex queries)
        if intent == "complex":
            sub_queries = self._decompose_query(query)
            docs = []
            for sq in sub_queries:
                docs.extend(self.retriever.similarity_search(sq, k=3))
        else:
            docs = self.retriever.similarity_search(query, k=5)
        
        # Stage 3: Re-ranking
        reranked_docs = self._rerank(query, docs)
        
        return {
            "query": query,
            "intent": intent,
            "context": "\n\n".join([d.page_content for d in reranked_docs[:5]]),
            "sources": [d.metadata for d in reranked_docs[:5]]
        }
    
    def _classify_intent(self, query: str) -> str:
        # Classify as "simple", "complex", "clarification", etc.
        return "simple"
    
    def _decompose_query(self, query: str) -> List[str]:
        # Break complex query into sub-queries
        return [query]
    
    def _rerank(self, query: str, docs: List) -> List:
        # Apply cross-encoder reranking
        return docs
```

## Testing Pyfunc Model Locally

```python
# Load and test
loaded_model = mlflow.pyfunc.load_model(f"runs:/{run_id}/rag_model")

test_input = pd.DataFrame({"query": [
    "What are the key features of Delta Lake?",
    "How does Vector Search work in Databricks?"
]})

predictions = loaded_model.predict(test_input)
for pred in predictions:
    print(f"Answer: {pred['answer'][:100]}...")
    print(f"Sources: {pred['sources']}")
```

## Best Practices

- **Clear separation**: Keep preprocessing, generation, postprocessing distinct for debugging.
- **Error handling**: Wrap each stage in try-except; return informative errors.
- **Caching**: Cache embeddings, retrieval results for repeated queries.
- **Versioning**: Log model version, dependencies, and config in MLflow.
- **Signature**: Define input/output schema for validation.
- **Artifacts**: Include retriever index paths, embedding model references.

## Sample Questions

1. What is a pyfunc model?
2. Why separate pre/post-processing?
3. How handle multiple input queries?
4. What goes in `load_context`?
5. How register pyfunc to Unity Catalog?

## Answers

1. MLflow Python function model; custom inference logic in `predict()` method.
2. Modularity, testability, easier debugging, reusable components.
3. Accept `pd.DataFrame` with multiple rows; iterate in `predict()`.
4. Load heavy resources (models, vector stores) once at deployment; store in `self`.
5. Use `mlflow.register_model()` with Unity Catalog namespace (e.g., `main.schema.model_name`).

## References

- [MLflow Pyfunc](https://mlflow.org/docs/latest/python_api/mlflow.pyfunc.html)
- [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html)

---

Next: [Vector Search Index Creation](../04-assembling-deploying-applications/02-vector-search-index-creation.md)
