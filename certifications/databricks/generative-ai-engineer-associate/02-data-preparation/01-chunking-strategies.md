# Chunking Strategies for Document Structure and Model Constraints

## Overview

Apply appropriate chunking strategies based on document structure (paragraphs, sections, tables) and model constraints (context window, token limits).

## Key Concepts

**Chunking**: Splitting documents into smaller segments for embedding and retrieval.

**Strategies**:

- **Fixed-size chunking**: Split by character/token count with overlap.
- **Semantic chunking**: Split by sentence, paragraph, or section boundaries.
- **Structure-aware chunking**: Preserve document hierarchy (headings, tables, code blocks).
- **Recursive chunking**: Split large chunks further until size constraints met.

## Chunk Size Considerations

| Model | Context Window | Recommended Chunk Size | Overlap |
|-------|---------------|----------------------|---------|
| BGE-large | 512 tokens | 256-400 tokens | 50-100 tokens |
| OpenAI text-embedding-3-large | 8191 tokens | 512-1024 tokens | 100-200 tokens |
| Llama 3.1 8B | 128K tokens | 512-2048 tokens | 100-300 tokens |

**Trade-offs**:

- **Small chunks** (128-256 tokens): Precise retrieval, more chunks to manage, potential context loss.
- **Large chunks** (1024-2048 tokens): More context, less precise, slower embedding.

## Hands-on Examples

### Fixed-Size Chunking

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,  # characters
    chunk_overlap=50,
    length_function=len,
    separators=["\n\n", "\n", ". ", " ", ""]
)

document = """
Apache Spark is a unified analytics engine for large-scale data processing.

Spark provides high-level APIs in Java, Scala, Python and R, and an optimized engine 
that supports general execution graphs.

It also supports a rich set of higher-level tools including Spark SQL for SQL and 
structured data processing, pandas API on Spark for pandas workloads, MLlib for machine 
learning, GraphX for graph processing, and Structured Streaming for incremental 
computation and stream processing.
"""

chunks = text_splitter.split_text(document)
print(f"Created {len(chunks)} chunks")
for i, chunk in enumerate(chunks):
    print(f"Chunk {i}: {len(chunk)} chars")
```

### Semantic Chunking by Paragraph

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

markdown_doc = """
# Data Engineering on Databricks

## Lakehouse Architecture

The lakehouse combines the best of data lakes and data warehouses. It provides ACID 
transactions, schema enforcement, and BI support directly on data lake storage.

## Delta Lake

Delta Lake is an open-source storage framework that brings reliability to data lakes. 
Key features include ACID transactions, time travel, and schema evolution.

### ACID Transactions

Delta Lake ensures data integrity with ACID properties: Atomicity, Consistency, 
Isolation, and Durability.
"""

headers_to_split_on = [
    ("#", "Header 1"),
    ("##", "Header 2"),
    ("###", "Header 3"),
]

markdown_splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
chunks = markdown_splitter.split_text(markdown_doc)

for chunk in chunks:
    print(f"Metadata: {chunk.metadata}")
    print(f"Content: {chunk.page_content[:100]}...")
```

### Token-Based Chunking for Specific Model

```python
from langchain.text_splitter import TokenTextSplitter
import tiktoken

# For OpenAI models
encoding = tiktoken.get_encoding("cl100k_base")

token_splitter = TokenTextSplitter(
    encoding_name="cl100k_base",
    chunk_size=512,  # tokens
    chunk_overlap=100
)

chunks = token_splitter.split_text(document)
```

### Structure-Aware Chunking for Code

```python
from langchain.text_splitter import Language, RecursiveCharacterTextSplitter

python_code = """
def process_data(df):
    '''Process DataFrame with transformations.'''
    # Filter active records
    active = df.filter(df.status == 'active')
    
    # Aggregate by region
    result = active.groupBy('region').agg(
        sum('revenue').alias('total_revenue')
    )
    
    return result

class DataPipeline:
    def __init__(self, spark):
        self.spark = spark
        
    def run(self):
        df = self.spark.read.parquet('/data/input')
        return process_data(df)
"""

python_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=200,
    chunk_overlap=20
)

code_chunks = python_splitter.split_text(python_code)
```

## Chunking for Different Document Types

### PDF with Tables

```python
from unstructured.partition.pdf import partition_pdf

# Extract with structure
elements = partition_pdf("document.pdf", strategy="hi_res")

chunks = []
for element in elements:
    if element.category == "Table":
        # Keep tables intact as single chunk
        chunks.append({
            "content": str(element),
            "type": "table",
            "metadata": element.metadata
        })
    elif element.category == "NarrativeText":
        # Split narrative by paragraph
        chunks.append({
            "content": element.text,
            "type": "text",
            "metadata": element.metadata
        })
```

### Recursive Chunking for Long Documents

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# First pass: split by major sections
section_splitter = RecursiveCharacterTextSplitter(
    chunk_size=2000,
    chunk_overlap=200,
    separators=["\n\n# ", "\n\n## ", "\n\n"]
)

# Second pass: further split large sections
final_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)

large_doc = open("large_document.txt").read()
sections = section_splitter.split_text(large_doc)

final_chunks = []
for section in sections:
    if len(section) > 500:
        final_chunks.extend(final_splitter.split_text(section))
    else:
        final_chunks.append(section)
```

## Best Practices

- Match chunk size to embedding model context window (leave headroom).
- Use semantic boundaries (sentences, paragraphs) over arbitrary character splits.
- Add overlap (10-20%) to preserve context across boundaries.
- Preserve structure for code, tables, and lists.
- Include metadata (source file, section heading, page number) with each chunk.
- Test retrieval quality with different chunk sizes; iterate.

## Sample Questions

1. Why use overlap in chunking?
2. Difference between fixed-size and semantic chunking?
3. Optimal chunk size for 512-token embedding model?
4. How chunk PDF with tables?
5. When use recursive chunking?

## Answers

1. Prevents information loss at boundaries; improves context continuity.
2. Fixed-size splits at arbitrary positions; semantic respects document structure (sentences, paragraphs).
3. 256-400 tokens with 50-100 token overlap.
4. Keep tables as single chunks; split narrative text separately; use structure-aware tools (Unstructured).
5. For long documents exceeding max chunk size; split in stages until constraints met.

## References

- [LangChain Text Splitters](https://python.langchain.com/docs/modules/data_connection/document_transformers/)
- [Unstructured.io](https://unstructured.io/)
- [Chunking Strategies Guide](https://www.pinecone.io/learn/chunking-strategies/)

---

Previous: [Multi-Stage Reasoning with Tools](../01-design-applications/05-multi-stage-reasoning-tools.md)  
Next: [Filtering Extraneous Content](./02-filtering-extraneous-content.md)
