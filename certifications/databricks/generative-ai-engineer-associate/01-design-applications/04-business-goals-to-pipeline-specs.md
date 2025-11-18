# Translating Business Goals to AI Pipeline Specifications

## Overview

Convert high-level business use cases into concrete input/output specifications for GenAI pipelines, defining data schemas, model requirements, and success criteria.

## Translation Framework

### Step 1: Identify Business Objective

- What problem are we solving?
- Who are the end users?
- What is the desired outcome?

### Step 2: Define Inputs

- Input data sources (structured, unstructured, real-time, batch).
- Input schema (fields, types, constraints).
- Input volume and frequency.

### Step 3: Define Outputs

- Output format (text, JSON, classification label, embedding).
- Output schema (keys, types, validation rules).
- Latency requirements (real-time <500ms, batch, async).

### Step 4: Map to AI Pipeline

- Model selection (task, size, hosting).
- Preprocessing steps (cleaning, chunking, embedding).
- Postprocessing (parsing, validation, formatting).
- Deployment architecture (API, batch, streaming).

## Hands-on Examples

### Use Case 1: Customer Support Ticket Routing

**Business Goal**: Automatically route incoming support tickets to correct department.

**Translation**:

```yaml
Input:
  - Source: Email system API
  - Schema:
      ticket_id: string
      subject: string
      body: string
      timestamp: datetime
  - Volume: ~500 tickets/day
  - Frequency: Real-time

Output:
  - Format: JSON
  - Schema:
      ticket_id: string
      department: enum [technical, billing, sales, general]
      confidence: float (0-1)
      reasoning: string (optional)
  - Latency: <2 seconds

AI Pipeline:
  1. Preprocessing: Clean email body, extract key phrases
  2. Model: Zero-shot classification (Llama 3 70B or fine-tuned BERT)
  3. Postprocessing: JSON parser, confidence threshold filter (>0.7)
  4. Deployment: REST API endpoint with Model Serving
```

### Use Case 2: Internal Knowledge Base Q&A

**Business Goal**: Enable employees to query company policies via chat interface.

**Translation**:

```python
# Input Spec
class QueryInput(BaseModel):
    user_id: str
    question: str
    session_id: Optional[str]
    
# Output Spec
class QueryOutput(BaseModel):
    answer: str
    sources: List[str]  # Document references
    confidence: float

# Pipeline Design
pipeline = {
    "preprocessing": [
        "Query rewriting for clarity",
        "Embedding generation (BGE-large)",
    ],
    "retrieval": [
        "Vector search (top-k=5 from Unity Catalog Vector Search)",
        "Reranking (cross-encoder)",
    ],
    "generation": [
        "RAG prompt with retrieved context",
        "LLM: Llama 3.1 70B Instruct",
    ],
    "postprocessing": [
        "Citation extraction",
        "Confidence scoring",
        "Safety filtering",
    ],
    "deployment": "Model Serving endpoint with chat interface"
}
```

### Use Case 3: Contract Review Automation

**Business Goal**: Extract key terms (parties, dates, obligations) from legal contracts.

**Translation**:

```yaml
Input:
  - Source: PDF contracts in cloud storage
  - Schema:
      contract_id: string
      file_path: string (S3/ADLS)
  - Volume: 50-100 contracts/week
  - Frequency: Batch (nightly)

Output:
  - Format: Delta table in Unity Catalog
  - Schema:
      contract_id: string
      parties: array<string>
      effective_date: date
      termination_date: date
      key_obligations: array<struct<party: string, obligation: string>>
      extraction_timestamp: timestamp
  - Latency: <10 min per contract

AI Pipeline:
  1. Preprocessing:
     - PDF extraction (PyPDF2, Unstructured)
     - Chunking (500 tokens, overlap 50)
  2. Model: GPT-4 or Llama 3.1 405B with structured prompt
  3. Postprocessing:
     - JSON validation (Pydantic)
     - Date parsing and normalization
  4. Deployment: Databricks Job (Delta Live Tables)
```

## Specification Template

```python
from pydantic import BaseModel, Field
from typing import List, Optional
from enum import Enum

class PipelineSpec(BaseModel):
    """AI Pipeline Specification Template"""
    
    # Business Context
    use_case: str
    stakeholders: List[str]
    success_metrics: List[str]  # e.g., "95% accuracy", "p95 latency <500ms"
    
    # Input Specification
    input_schema: dict
    input_source: str
    input_volume: str
    input_frequency: str  # "real-time", "batch", "streaming"
    
    # Output Specification
    output_schema: dict
    output_format: str  # "json", "text", "table"
    latency_requirement: str
    
    # Model Requirements
    model_task: str  # "classification", "generation", "extraction"
    model_constraints: dict  # {"max_tokens": 2048, "temperature": 0.1}
    
    # Pipeline Components
    preprocessing_steps: List[str]
    retrieval_strategy: Optional[str]
    postprocessing_steps: List[str]
    
    # Deployment
    deployment_type: str  # "api", "batch", "streaming"
    infrastructure: str  # "model_serving", "databricks_job"
```

## Best Practices

- Involve domain experts to validate input/output schemas.
- Define measurable success criteria upfront (accuracy, latency, cost).
- Start with minimum viable pipeline; iterate based on feedback.
- Document assumptions (data quality, input distribution, edge cases).
- Plan for monitoring and retraining from day one.

## Sample Questions

1. How translate "classify emails" to pipeline spec?
2. Key inputs for a RAG knowledge base?
3. Output schema for document summarization?
4. Latency requirement for real-time chatbot?
5. When use batch vs real-time deployment?

## Answers

1. Input: email text; Output: category label + confidence; Model: classification (BERT or GPT).
2. User query, document corpus (embedded), retrieval config (top-k), LLM for generation.
3. JSON with keys: `summary` (string), `original_length` (int), `compression_ratio` (float).
4. Typically <500ms end-to-end for acceptable UX.
5. Batch for high-volume offline processing; real-time for user-facing apps requiring immediate response.

## References

- [Databricks GenAI Solutions](https://docs.databricks.com/en/generative-ai/index.html)
- [Pydantic for Schema Validation](https://docs.pydantic.dev/)
- [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html)

---

Previous: [Chain Component Selection](./03-chain-component-selection.md)  
Next: [Multi-Stage Reasoning with Tools](./05-multi-stage-reasoning-tools.md)
