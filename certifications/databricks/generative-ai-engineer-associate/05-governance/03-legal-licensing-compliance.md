# Using Legal and Licensing Requirements to Avoid Legal Risk

## Overview

Navigate legal and licensing considerations for GenAI applications: data usage rights, model licensing, compliance requirements, and intellectual property protection.

## Legal Risk Areas

| Risk Category | Considerations | Mitigation |
|---------------|----------------|------------|
| **Data Rights** | Can you use data for training/RAG? | Audit data sources, obtain consent |
| **Model Licensing** | Open-source vs commercial restrictions | Review license terms (Apache 2.0, MIT, proprietary) |
| **IP Infringement** | Generated content copyright | Implement attribution, check for copied content |
| **Compliance** | GDPR, CCPA, HIPAA, SOC 2 | Data governance, audit trails |
| **Liability** | Harmful outputs | Disclaimers, guardrails, human review |

## Hands-on Examples

### Data Source Audit Framework

```python
from dataclasses import dataclass
from typing import List
from enum import Enum

class DataLicenseType(Enum):
    PUBLIC_DOMAIN = "public_domain"
    CREATIVE_COMMONS = "creative_commons"
    PROPRIETARY_LICENSED = "proprietary_licensed"
    INTERNAL = "internal"
    UNKNOWN = "unknown"

@dataclass
class DataSource:
    name: str
    license_type: DataLicenseType
    license_terms: str
    can_use_for_training: bool
    can_use_for_rag: bool
    can_share_publicly: bool
    attribution_required: bool
    commercial_use_allowed: bool
    expiration_date: str = None

class DataSourceAuditor:
    def __init__(self):
        self.approved_sources = []
    
    def audit_source(self, source: DataSource) -> dict:
        """Audit data source for legal compliance."""
        
        issues = []
        
        # Check if license known
        if source.license_type == DataLicenseType.UNKNOWN:
            issues.append("License type unknown - BLOCKER")
        
        # Check RAG usage rights
        if not source.can_use_for_rag:
            issues.append("Not licensed for RAG use")
        
        # Check commercial use
        if not source.commercial_use_allowed:
            issues.append("Commercial use not allowed")
        
        # Check expiration
        if source.expiration_date:
            from datetime import datetime
            expiry = datetime.fromisoformat(source.expiration_date)
            if expiry < datetime.now():
                issues.append("License EXPIRED")
        
        return {
            "source": source.name,
            "compliant": len(issues) == 0,
            "issues": issues,
            "requires_attribution": source.attribution_required
        }

# Example usage
sources = [
    DataSource(
        name="Wikipedia",
        license_type=DataLicenseType.CREATIVE_COMMONS,
        license_terms="CC BY-SA 4.0",
        can_use_for_training=True,
        can_use_for_rag=True,
        can_share_publicly=True,
        attribution_required=True,
        commercial_use_allowed=True
    ),
    DataSource(
        name="Internal Customer Support Logs",
        license_type=DataLicenseType.INTERNAL,
        license_terms="Internal use only",
        can_use_for_training=True,
        can_use_for_rag=True,
        can_share_publicly=False,
        attribution_required=False,
        commercial_use_allowed=True
    ),
    DataSource(
        name="Third-Party Research Papers",
        license_type=DataLicenseType.PROPRIETARY_LICENSED,
        license_terms="Licensed for internal use, no redistribution",
        can_use_for_training=False,
        can_use_for_rag=True,
        can_share_publicly=False,
        attribution_required=True,
        commercial_use_allowed=False
    )
]

auditor = DataSourceAuditor()
for source in sources:
    result = auditor.audit_source(source)
    print(f"{result['source']}: {'✓ Compliant' if result['compliant'] else '✗ Issues'}")
    if result['issues']:
        for issue in result['issues']:
            print(f"  - {issue}")
```

### Model License Compliance Check

```python
@dataclass
class ModelLicense:
    model_name: str
    license: str  # e.g., "Apache 2.0", "MIT", "Llama 3 Community License"
    commercial_use_allowed: bool
    modification_allowed: bool
    distribution_allowed: bool
    attribution_required: bool
    restrictions: List[str]

APPROVED_MODELS = [
    ModelLicense(
        model_name="Llama 3.1",
        license="Llama 3.1 Community License",
        commercial_use_allowed=True,
        modification_allowed=True,
        distribution_allowed=True,
        attribution_required=True,
        restrictions=["Must comply with Acceptable Use Policy"]
    ),
    ModelLicense(
        model_name="DBRX",
        license="Databricks Open Model License",
        commercial_use_allowed=True,
        modification_allowed=True,
        distribution_allowed=True,
        attribution_required=True,
        restrictions=[]
    ),
    ModelLicense(
        model_name="BGE-Large-EN",
        license="MIT",
        commercial_use_allowed=True,
        modification_allowed=True,
        distribution_allowed=True,
        attribution_required=True,
        restrictions=[]
    )
]

def check_model_compliance(model_name: str, use_case: str) -> dict:
    """Check if model usage complies with license."""
    
    model_license = next((m for m in APPROVED_MODELS if m.model_name == model_name), None)
    
    if not model_license:
        return {"compliant": False, "reason": "Model not in approved list"}
    
    # Check commercial use
    if use_case == "commercial" and not model_license.commercial_use_allowed:
        return {"compliant": False, "reason": "Commercial use not permitted"}
    
    return {
        "compliant": True,
        "attribution_required": model_license.attribution_required,
        "restrictions": model_license.restrictions
    }
```

### Attribution Generator for Outputs

```python
def generate_attribution(sources: List[dict]) -> str:
    """Generate proper attribution for RAG responses."""
    
    attributions = []
    
    for source in sources:
        if source.get('attribution_required'):
            citation = f"[{source['id']}] {source['title']}"
            
            # Add license info if required
            if source.get('license'):
                citation += f" (Licensed under {source['license']})"
            
            # Add URL if available
            if source.get('url'):
                citation += f" - {source['url']}"
            
            attributions.append(citation)
    
    if attributions:
        return "\n\nSources:\n" + "\n".join(attributions)
    return ""

# Usage in RAG response
def rag_with_attribution(query: str) -> str:
    # Retrieve documents
    docs = retriever.get_relevant_documents(query)
    
    # Generate answer
    answer = llm.invoke(query_with_context).content
    
    # Add attribution
    attribution = generate_attribution([
        {
            "id": 1,
            "title": "Unity Catalog Documentation",
            "license": "CC BY 4.0",
            "url": "https://docs.databricks.com/unity-catalog",
            "attribution_required": True
        }
    ])
    
    return answer + attribution
```

### Compliance Metadata Tracking

```python
# Track compliance metadata in Delta tables

compliance_schema = """
    document_id STRING,
    source STRING,
    license_type STRING,
    can_use_for_rag BOOLEAN,
    pii_detected BOOLEAN,
    data_classification STRING,
    consent_obtained BOOLEAN,
    retention_days INT,
    last_audit_date DATE
"""

# Example: Audit document before adding to RAG
def audit_document_for_rag(document_id: str, content: str, metadata: dict) -> dict:
    """Ensure document meets legal requirements."""
    
    checks = {
        "license_approved": metadata.get('license') in ['CC BY', 'CC BY-SA', 'Internal'],
        "no_pii": not contains_pii(content),
        "consent_obtained": metadata.get('consent_obtained', False),
        "retention_valid": metadata.get('retention_days', 0) > 0
    }
    
    all_passed = all(checks.values())
    
    # Log audit
    audit_record = {
        "document_id": document_id,
        "timestamp": datetime.now(),
        "checks": checks,
        "compliant": all_passed
    }
    
    return audit_record

def contains_pii(text: str) -> bool:
    # Use Presidio or similar
    from presidio_analyzer import AnalyzerEngine
    analyzer = AnalyzerEngine()
    results = analyzer.analyze(text=text, language='en')
    return len(results) > 0
```

### GDPR/CCPA Right to Deletion

```python
def delete_user_data(user_id: str):
    """Implement right to deletion (GDPR Article 17)."""
    
    from delta.tables import DeltaTable
    
    # Delete from vector index source table
    documents_table = DeltaTable.forName(spark, "main.rag_demo.document_chunks")
    documents_table.delete(f"metadata.user_id = '{user_id}'")
    
    # Delete from inference logs
    inference_table = DeltaTable.forName(spark, "main.monitoring.inference_logs")
    inference_table.delete(f"user_id = '{user_id}'")
    
    # Trigger vector index re-sync
    from databricks.sdk import WorkspaceClient
    w = WorkspaceClient()
    w.vector_search_indexes.sync_index("main.rag_demo.docs_vector_index")
    
    # Log deletion for audit trail
    deletion_log = {
        "user_id": user_id,
        "deletion_timestamp": datetime.now(),
        "tables_affected": ["document_chunks", "inference_logs"]
    }
    
    spark.createDataFrame([deletion_log]).write.mode("append").saveAsTable(
        "main.compliance.deletion_log"
    )
```

### Output Copyright Check

```python
def check_for_copied_content(generated_text: str, source_docs: List[str]) -> dict:
    """Detect if output is substantially copied from sources."""
    
    from difflib import SequenceMatcher
    
    max_similarity = 0
    most_similar_source = None
    
    for source in source_docs:
        similarity = SequenceMatcher(None, generated_text, source).ratio()
        if similarity > max_similarity:
            max_similarity = similarity
            most_similar_source = source
    
    # Flag if >80% similar (potential copyright issue)
    is_copied = max_similarity > 0.8
    
    return {
        "is_potential_copy": is_copied,
        "max_similarity": max_similarity,
        "source": most_similar_source[:100] if is_copied else None,
        "recommendation": "Regenerate with paraphrasing" if is_copied else "OK to use"
    }
```

## Best Practices

- **License audit**: Catalog all data sources with license terms before use.
- **Attribution by default**: Always include source citations when required.
- **Consent tracking**: Log user consent for data usage in metadata.
- **Retention policies**: Implement automated data deletion per compliance.
- **Regular reviews**: Annual legal review of data sources and licenses.
- **Disclaimers**: Include clear disclaimers for AI-generated content.

## Sample Questions

1. What is Creative Commons license?
2. How handle GDPR right to deletion?
3. When required to attribute sources?
4. Can you use copyrighted data for RAG?
5. Best practice for model licensing?

## Answers

1. Open licenses allowing reuse with conditions (BY=attribution, SA=share-alike, NC=non-commercial); CC BY-SA allows commercial use with attribution.
2. Implement deletion from all tables (source, embeddings, logs), trigger vector index re-sync, maintain audit log of deletions.
3. When data license requires (e.g., CC BY), when using proprietary licensed content, or when beneficial for transparency.
4. Only with explicit license allowing it; fair use is narrow and risky; best to use openly licensed or internal data.
5. Use approved models (Apache 2.0, MIT, Llama Community License), document license terms, ensure commercial use allowed if needed, maintain attribution.

## References

- [Creative Commons Licenses](https://creativecommons.org/licenses/)
- [GDPR Article 17 (Right to Erasure)](https://gdpr-info.eu/art-17-gdpr/)
- [Llama 3 License](https://www.llama.com/llama3/license/)

---

Previous: [Protecting Against Malicious Inputs](./02-protecting-malicious-inputs.md)  
Next: [RAG Evaluation with MLflow](../06-evaluation-monitoring/01-rag-evaluation-mlflow.md)
