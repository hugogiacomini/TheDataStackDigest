# Writing Chunked Text to Delta Lake in Unity Catalog

## Overview

Define operations and sequence to write chunked text and embeddings into Delta tables in Unity Catalog for RAG applications.

## Key Concepts

**Delta Lake**: ACID-compliant storage layer for data lakes.

**Unity Catalog**: Unified governance for data and AI assets on Databricks.

**Typical Schema**:

```python
{
    "chunk_id": "string (UUID)",
    "content": "string (chunked text)",
    "embedding": "array<double> (vector)",
    "source_file": "string",
    "chunk_index": "int",
    "metadata": "map<string,string>",
    "created_at": "timestamp"
}
```

## Hands-on Examples

### Create Delta Table

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, ArrayType, DoubleType, MapType, TimestampType
from delta.tables import DeltaTable

spark = SparkSession.builder.getOrCreate()

schema = StructType([
    StructField("chunk_id", StringType(), False),
    StructField("content", StringType(), False),
    StructField("embedding", ArrayType(DoubleType()), True),
    StructField("source_file", StringType(), False),
    StructField("chunk_index", IntegerType(), False),
    StructField("metadata", MapType(StringType(), StringType()), True),
    StructField("created_at", TimestampType(), False)
])

# Create empty Delta table in Unity Catalog
(spark.createDataFrame([], schema)
    .write
    .format("delta")
    .mode("overwrite")
    .option("delta.enableChangeDataFeed", "true")
    .saveAsTable("main.rag_demo.document_chunks"))
```

### Write Chunks to Delta

```python
import uuid
from datetime import datetime
from typing import List

def write_chunks_to_delta(chunks: List[str], source_file: str, catalog="main", schema="rag_demo", table="document_chunks"):
    """Write chunks to Delta table."""
    
    # Prepare data
    data = []
    for idx, chunk in enumerate(chunks):
        data.append({
            "chunk_id": str(uuid.uuid4()),
            "content": chunk,
            "embedding": None,  # Will be populated later
            "source_file": source_file,
            "chunk_index": idx,
            "metadata": {"doc_type": "pdf", "processed_date": datetime.now().isoformat()},
            "created_at": datetime.now()
        })
    
    # Create DataFrame
    df = spark.createDataFrame(data)
    
    # Write to Delta (append mode)
    (df.write
        .format("delta")
        .mode("append")
        .saveAsTable(f"{catalog}.{schema}.{table}"))
    
    print(f"Wrote {len(chunks)} chunks from {source_file}")

# Example usage
chunks = [
    "Apache Spark is a unified analytics engine...",
    "Delta Lake provides ACID transactions...",
]
write_chunks_to_delta(chunks, "spark_docs.pdf")
```

### Generate and Update Embeddings

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import EmbeddingsV1QueryInput
import numpy as np

w = WorkspaceClient()

def generate_embeddings(texts: List[str], endpoint="databricks-bge-large-en") -> List[List[float]]:
    """Generate embeddings using Databricks endpoint."""
    
    response = w.serving_endpoints.query(
        name=endpoint,
        inputs=texts
    )
    
    return [embedding for embedding in response.data]

# Read chunks without embeddings
chunks_df = spark.table("main.rag_demo.document_chunks").filter("embedding IS NULL")

# Generate embeddings in batches
from pyspark.sql.functions import pandas_udf, col
import pandas as pd

@pandas_udf("array<double>")
def embed_text(texts: pd.Series) -> pd.Series:
    batch = texts.tolist()
    embeddings = generate_embeddings(batch)
    return pd.Series(embeddings)

# Update with embeddings
updated_df = chunks_df.withColumn("embedding", embed_text(col("content")))

(updated_df.write
    .format("delta")
    .mode("overwrite")
    .option("replaceWhere", "embedding IS NULL")
    .saveAsTable("main.rag_demo.document_chunks"))
```

### Incremental Processing with Delta Live Tables

```python
import dlt
from pyspark.sql.functions import col, current_timestamp

@dlt.table(
    name="bronze_documents",
    comment="Raw document chunks"
)
def bronze_layer():
    return (
        spark.readStream
            .format("cloudFiles")
            .option("cloudFiles.format", "json")
            .schema("content STRING, source STRING")
            .load("/mnt/raw/documents/")
            .withColumn("ingestion_time", current_timestamp())
    )

@dlt.table(
    name="silver_chunks",
    comment="Cleaned and chunked documents"
)
def silver_layer():
    # Apply chunking logic
    return dlt.read_stream("bronze_documents").selectExpr("*", "chunk_text(content) as chunks")

@dlt.table(
    name="gold_embeddings",
    comment="Chunks with embeddings"
)
def gold_layer():
    return (
        dlt.read_stream("silver_chunks")
            .withColumn("embedding", embed_text(col("chunks")))
    )
```

## Operation Sequence

1. **Ingest raw documents** → Bronze table (raw text, metadata).
2. **Clean and chunk** → Silver table (filtered, chunked).
3. **Generate embeddings** → Gold table (chunks + vectors).
4. **Create Vector Search index** → Enable similarity search.

## Best Practices

- Use **Delta Change Data Feed** for tracking updates.
- Partition by `source_file` or `created_at` for efficient queries.
- Add **constraints** for data quality: `NOT NULL` on key fields.
- Use **Z-ORDER** on frequently queried columns (e.g., `source_file`).
- Enable **liquid clustering** for large tables (Databricks Runtime 13.3+).
- **Version tables** with Delta time travel for rollback.
- Use **Unity Catalog tags** for governance (PII, sensitivity).

## Sample Questions

1. Why use Delta for RAG applications?
2. Required fields in chunk table schema?
3. How update embeddings incrementally?
4. When use Delta Live Tables?
5. Best partition strategy for document chunks?

## Answers

1. ACID guarantees, time travel, efficient updates/merges, Unity Catalog integration.
2. `chunk_id` (unique), `content` (text), `embedding` (vector), `source_file`, `chunk_index`.
3. Filter `WHERE embedding IS NULL`, generate embeddings, merge/overwrite with `replaceWhere`.
4. For streaming/incremental pipelines with data quality checks and lineage tracking.
5. Partition by `source_file` for document-level operations; by `created_at` (date) for time-based queries.

## References

- [Delta Lake](https://docs.delta.io/)
- [Unity Catalog](https://docs.databricks.com/en/data-governance/unity-catalog/index.html)
- [Delta Live Tables](https://docs.databricks.com/en/delta-live-tables/index.html)

---

Previous: [Document Content Extraction](./03-document-content-extraction.md)  
Next: [Source Document Selection for RAG](./05-source-document-selection.md)
