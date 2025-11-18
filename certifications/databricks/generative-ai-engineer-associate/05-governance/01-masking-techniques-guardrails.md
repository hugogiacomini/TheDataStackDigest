# Masking Techniques as Guardrails for Performance Objectives

## Overview

Apply data masking (PII redaction, entity replacement, differential privacy) to protect sensitive information while maintaining RAG application functionality and compliance.

## Masking Strategies

| Technique | Use Case | Example |
|-----------|----------|---------|
| **Redaction** | Remove sensitive data completely | `John Doe` → `[REDACTED]` |
| **Pseudonymization** | Replace with fake but consistent values | `john.doe@company.com` → `user_12345@example.com` |
| **Generalization** | Reduce precision | `$75,432` → `$70,000-$80,000` |
| **Tokenization** | Replace with reversible tokens | `SSN:123-45-6789` → `TOKEN_A7B3C` |
| **Differential Privacy** | Add noise to aggregates | `Average: 45.2` → `Average: 45.2 ± ε` |

## Hands-on Examples

### PII Masking with Presidio

```python
from presidio_analyzer import AnalyzerEngine
from presidio_anonymizer import AnonymizerEngine
from presidio_anonymizer.entities import OperatorConfig

analyzer = AnalyzerEngine()
anonymizer = AnonymizerEngine()

def mask_pii(text: str, mask_type="replace") -> str:
    """Mask PII in text."""
    
    # Detect PII
    results = analyzer.analyze(text=text, language='en')
    
    # Define masking operators
    operators = {
        "PERSON": OperatorConfig("replace", {"new_value": "<PERSON>"}),
        "EMAIL_ADDRESS": OperatorConfig("replace", {"new_value": "<EMAIL>"}),
        "PHONE_NUMBER": OperatorConfig("replace", {"new_value": "<PHONE>"}),
        "CREDIT_CARD": OperatorConfig("mask", {"masking_char": "*", "chars_to_mask": 12, "from_end": False}),
        "US_SSN": OperatorConfig("replace", {"new_value": "<SSN>"}),
    }
    
    # Apply masking
    anonymized = anonymizer.anonymize(text=text, analyzer_results=results, operators=operators)
    return anonymized.text

# Example
sensitive_text = """
Customer John Doe (john.doe@company.com, phone: 555-123-4567) 
reported issue with card ending in 4532-1234-5678-9010.
SSN on file: 123-45-6789.
"""

masked = mask_pii(sensitive_text)
print(masked)
# Output: Customer <PERSON> (<EMAIL>, phone: <PHONE>) reported issue with card ending in ****-****-****-9010. SSN on file: <SSN>.
```

### Context-Preserving Masking

```python
import hashlib

def pseudonymize_consistent(text: str, entity_type: str) -> str:
    """Replace entity with consistent pseudonym."""
    
    # Generate deterministic pseudonym
    hash_val = hashlib.md5(text.encode()).hexdigest()[:8]
    
    pseudonym_map = {
        "PERSON": f"Person_{hash_val}",
        "EMAIL": f"user_{hash_val}@example.com",
        "PHONE": f"555-{hash_val[:3]}-{hash_val[3:7]}"
    }
    
    return pseudonym_map.get(entity_type, f"<{entity_type}>")

def mask_with_context_preservation(text: str) -> str:
    """Mask while preserving semantic relationships."""
    
    analyzer = AnalyzerEngine()
    results = analyzer.analyze(text=text, language='en')
    
    # Build mapping for consistency
    entity_map = {}
    
    masked_text = text
    for result in sorted(results, key=lambda x: x.start, reverse=True):
        entity_text = text[result.start:result.end]
        
        if entity_text not in entity_map:
            entity_map[entity_text] = pseudonymize_consistent(entity_text, result.entity_type)
        
        masked_text = masked_text[:result.start] + entity_map[entity_text] + masked_text[result.end:]
    
    return masked_text, entity_map
```

### RAG Pipeline with Masking

```python
class MaskedRAGPipeline:
    def __init__(self, vectorstore, llm):
        self.vectorstore = vectorstore
        self.llm = llm
        self.masker = AnonymizerEngine()
    
    def query(self, user_query: str) -> dict:
        """Process query with PII masking."""
        
        # 1. Mask user query
        masked_query, query_entity_map = self._mask_text(user_query)
        
        # 2. Retrieve (using masked query)
        docs = self.vectorstore.similarity_search(masked_query, k=5)
        context = "\n".join([doc.page_content for doc in docs])
        
        # 3. Mask retrieved context
        masked_context, context_entity_map = self._mask_text(context)
        
        # 4. Generate response
        prompt = f"""Answer using only the context provided.

Context: {masked_context}

Question: {masked_query}

Answer:"""
        
        response = self.llm.invoke(prompt).content
        
        # 5. Optional: Unmask response if needed (with proper access controls)
        # unmasked_response = self._unmask_text(response, combined_entity_map)
        
        return {
            "response": response,
            "masked": True,
            "entity_types_detected": list(set([r.entity_type for r in analyzer.analyze(user_query, 'en')]))
        }
    
    def _mask_text(self, text: str) -> tuple:
        results = analyzer.analyze(text=text, language='en')
        anonymized = self.masker.anonymize(text=text, analyzer_results=results)
        return anonymized.text, results
```

### Databricks Unity Catalog Column-Level Masking

```python
# Apply masking at table level using Unity Catalog

# SQL approach
spark.sql("""
CREATE OR REPLACE TABLE main.secure.customer_data (
  customer_id STRING,
  name STRING MASK hash(name) EXCEPT (role='analyst'),
  email STRING MASK mask(email) EXCEPT (role='admin'),
  purchase_amount DECIMAL(10,2)
)
""")

# Python approach with dynamic column masking
from pyspark.sql.functions import when, col, lit

def apply_column_masking(df, user_role):
    """Apply role-based masking."""
    
    if user_role == "admin":
        return df
    elif user_role == "analyst":
        return df.withColumn("email", lit("<MASKED>"))
    else:
        return df.select("customer_id", "purchase_amount")  # Minimal columns
```

## Governance Integration

```python
def masked_rag_with_audit_log(query: str, user_id: str, user_role: str):
    """RAG with masking and audit trail."""
    
    # Check permissions
    if not can_access_pii(user_role):
        query = mask_pii(query)
    
    # Log access
    log_audit_event({
        "user_id": user_id,
        "query": query if can_access_pii(user_role) else "<MASKED>",
        "pii_accessed": contains_pii(query),
        "timestamp": datetime.now()
    })
    
    # Execute query
    response = rag_pipeline.query(query)
    return response
```

## Best Practices

- **Mask early**: Apply at ingestion, not just at query time.
- **Audit masking decisions**: Log what was masked and why.
- **Role-based masking**: Different roles see different levels of detail.
- **Test thoroughly**: Ensure masking doesn't break semantic understanding.
- **Reversibility**: Use tokenization when unmasking may be needed (with proper controls).
- **Performance**: Cache masked versions to avoid repeated computation.

## Sample Questions

1. Difference between redaction and pseudonymization?
2. When preserve entity relationships during masking?
3. How mask PII in Delta tables?
4. Trade-off of aggressive masking in RAG?
5. Best practice for email masking?

## Answers

1. Redaction removes completely; pseudonymization replaces with fake but consistent values (preserves analysis).
2. When downstream logic depends on entity co-occurrence or frequency (e.g., "same customer contacted twice").
3. Unity Catalog column masks, or apply transformation during write with PySpark UDFs.
4. May degrade retrieval/generation quality if context lost; balance with risk tolerance.
5. Replace with `<EMAIL>` or pseudonymize (`user_XXX@example.com`) to preserve email-ness for format validation.

## References

- [Microsoft Presidio](https://microsoft.github.io/presidio/)
- [Unity Catalog Data Masking](https://docs.databricks.com/en/data-governance/unity-catalog/manage-privileges/column-masks.html)

---

Previous: [Vector Search Index Creation](../04-assembling-deploying-applications/02-vector-search-index-creation.md)  
Next: [RAG Evaluation with MLflow](../06-evaluation-monitoring/01-rag-evaluation-mlflow.md)
