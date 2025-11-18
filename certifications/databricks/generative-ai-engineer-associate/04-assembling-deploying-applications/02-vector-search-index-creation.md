# Vector Search Index Creation and Querying

## Overview

Create, manage, and query Databricks Vector Search indexes for semantic similarity search in RAG applications.

## Vector Search Architecture

```text
Delta Table (chunks + embeddings) → Vector Search Index → Similarity Queries
```

**Index Types**:

- **Delta Sync Index**: Auto-syncs with Delta table; managed embeddings.
- **Direct Vector Access Index**: Bring your own embeddings.

## Hands-on Examples

### Create Vector Search Endpoint

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.vectorsearch import EndpointType

w = WorkspaceClient()

# Create endpoint (one-time setup)
endpoint_name = "rag_vector_search_endpoint"

w.vector_search_endpoints.create_endpoint(
    name=endpoint_name,
    endpoint_type=EndpointType.STANDARD
)

# Wait for endpoint to be ready
w.vector_search_endpoints.wait_get_endpoint_vector_search_endpoint_online(endpoint_name)
```

### Create Delta Sync Index

```python
from databricks.sdk.service.vectorsearch import (
    DeltaSyncVectorIndexSpecRequest,
    EmbeddingSourceColumn,
    PipelineType
)

# Assuming Delta table: main.rag_demo.document_chunks
# Schema: chunk_id, content, embedding (array<double>), source_file, metadata

index_name = "main.rag_demo.docs_vector_index"

w.vector_search_indexes.create_index(
    name=index_name,
    endpoint_name=endpoint_name,
    primary_key="chunk_id",
    index_type="DELTA_SYNC",
    delta_sync_index_spec=DeltaSyncVectorIndexSpecRequest(
        source_table="main.rag_demo.document_chunks",
        embedding_source_columns=[
            EmbeddingSourceColumn(
                name="content",
                embedding_model_endpoint_name="databricks-bge-large-en"
            )
        ],
        pipeline_type=PipelineType.TRIGGERED  # or CONTINUOUS
    )
)

# Monitor index creation
index_status = w.vector_search_indexes.get_index(index_name)
print(f"Index status: {index_status.status.detailed_state}")
```

### Create Direct Vector Access Index

```python
# For pre-computed embeddings in Delta table

w.vector_search_indexes.create_index(
    name="main.rag_demo.precomputed_embeddings_index",
    endpoint_name=endpoint_name,
    primary_key="chunk_id",
    index_type="DIRECT_ACCESS",
    direct_access_index_spec={
        "embedding_source_columns": [{
            "embedding_dimension": 1024,  # BGE-large dimension
            "name": "embedding"
        }],
        "schema_json": {
            "columns": [
                {"name": "chunk_id", "type": "string"},
                {"name": "content", "type": "string"},
                {"name": "embedding", "type": "array<double>"}
            ]
        }
    }
)
```

### Query Vector Search Index

```python
# Using SDK
query_text = "What are the benefits of Delta Lake?"

results = w.vector_search_indexes.query_index(
    index_name="main.rag_demo.docs_vector_index",
    query_text=query_text,
    columns=["chunk_id", "content", "source_file"],
    num_results=5
)

for item in results.result.data_array:
    chunk_id, content, source = item
    print(f"Source: {source}")
    print(f"Content: {content[:200]}...")
    print("---")
```

### Query with Filters

```python
# Filter by metadata
results = w.vector_search_indexes.query_index(
    index_name="main.rag_demo.docs_vector_index",
    query_text=query_text,
    columns=["chunk_id", "content", "source_file", "metadata"],
    filters={"source_file": ("=", "databricks_docs.pdf")},
    num_results=10
)
```

### LangChain Integration

```python
from langchain_community.vectorstores import DatabricksVectorSearch
from langchain_community.embeddings import DatabricksEmbeddings

embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")

vectorstore = DatabricksVectorSearch(
    endpoint=endpoint_name,
    index_name="main.rag_demo.docs_vector_index",
    embedding=embeddings,
    text_column="content",
    columns=["chunk_id", "content", "source_file"]
)

# Use as retriever
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

docs = retriever.get_relevant_documents("How does Unity Catalog work?")
for doc in docs:
    print(doc.page_content[:150])
    print(doc.metadata)
```

### Update Index (Add New Documents)

```python
# For Delta Sync indexes, just append to source Delta table

new_chunks = spark.createDataFrame([
    {"chunk_id": "new_001", "content": "New document content...", "source_file": "new_doc.pdf"}
])

new_chunks.write.format("delta").mode("append").saveAsTable("main.rag_demo.document_chunks")

# Trigger index sync
w.vector_search_indexes.sync_index("main.rag_demo.docs_vector_index")
```

## Index Management

### Monitor Index Status

```python
index_info = w.vector_search_indexes.get_index("main.rag_demo.docs_vector_index")
print(f"Status: {index_info.status.detailed_state}")
print(f"Indexed rows: {index_info.status.indexed_row_count}")
```

### Delete Index

```python
w.vector_search_indexes.delete_index("main.rag_demo.docs_vector_index")
```

## Best Practices

- **Endpoint sizing**: Use STANDARD for production; scale based on QPS.
- **Index updates**: TRIGGERED for batch; CONTINUOUS for real-time.
- **Embedding consistency**: Use same embedding model for indexing and querying.
- **Filters**: Add filters (metadata) for multi-tenant or scoped search.
- **Monitoring**: Track query latency, index freshness in production.
- **Chunking alignment**: Ensure chunk size matches embedding model context.

## Sample Questions

1. Difference between Delta Sync and Direct Access indexes?
2. How add new documents to index?
3. What is a Vector Search endpoint?
4. How filter query results?
5. Best index type for real-time updates?

## Answers

1. Delta Sync auto-syncs with Delta table, manages embeddings; Direct Access uses pre-computed embeddings.
2. Append to source Delta table; trigger sync for Delta Sync indexes.
3. Compute resource hosting vector search operations; supports multiple indexes.
4. Use `filters` parameter in `query_index` with metadata column conditions.
5. Delta Sync with CONTINUOUS pipeline for auto-updates; Direct Access requires manual refresh.

## References

- [Databricks Vector Search](https://docs.databricks.com/en/generative-ai/vector-search.html)
- [Vector Search API](https://docs.databricks.com/api/workspace/vectorsearchindexes)

---

Previous: [Pyfunc Models for RAG](./01-pyfunc-models-rag.md)  
Next: [Governance: Masking Techniques](../05-governance/01-masking-techniques-guardrails.md)
