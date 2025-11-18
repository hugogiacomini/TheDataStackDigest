# Source Document Selection for RAG Quality

## Overview

Identify and curate source documents that provide necessary knowledge, accuracy, and quality for RAG applications.

## Selection Criteria

1. **Relevance**: Content aligns with application domain and user queries.
2. **Authoritativeness**: Trusted, verified sources (official docs, subject matter experts).
3. **Recency**: Up-to-date information (critical for fast-changing domains).
4. **Completeness**: Comprehensive coverage of topic without gaps.
5. **Quality**: Well-written, factual, free of errors and bias.
6. **Format**: Extractable structure (avoid scanned images without OCR).

## Document Sourcing Strategies

### Internal Knowledge Bases

- Technical documentation (Confluence, SharePoint, wikis)
- Policy manuals, SOPs, guidelines
- Historical support tickets and resolutions
- Product specs, design docs

### External Sources

- Official vendor documentation
- Industry standards and whitepapers
- Academic papers and research
- Regulatory guidelines

### Quality Assessment

```python
def assess_document_quality(doc: str) -> dict:
    """Score document on multiple quality dimensions."""
    
    scores = {}
    
    # Readability (Flesch-Kincaid)
    import textstat
    scores['readability'] = textstat.flesch_reading_ease(doc)
    
    # Completeness (length, structure)
    scores['word_count'] = len(doc.split())
    scores['has_structure'] = any(marker in doc for marker in ['#', '##', '1.', '2.'])
    
    # Recency (requires metadata)
    # scores['days_old'] = (datetime.now() - doc_created_date).days
    
    # Authority (requires metadata)
    # scores['source_trust_score'] = trust_scores.get(doc_source, 0)
    
    return scores

# Filter documents
good_docs = [doc for doc in candidates if assess_document_quality(doc)['word_count'] > 100]
```

## Sample Questions

1. Why prioritize authoritative sources?
2. How assess document recency?
3. When include external vs internal docs?
4. Red flags in source documents?
5. How validate document relevance?

## Answers

1. Reduces hallucinations and improves factual accuracy; builds user trust.
2. Check metadata timestamps; validate against known updates; penalize outdated content in retrieval.
3. Internal for proprietary knowledge, policies; external for industry standards, general concepts.
4. Contradictions, outdated info, poor formatting, excessive marketing language, missing citations.
5. Pilot with sample queries; measure retrieval precision; gather user feedback on answer quality.

## References

- [Document Selection Best Practices](https://www.pinecone.io/learn/rag/)

---

Previous: [Writing Chunks to Delta Lake](./04-writing-chunks-to-delta-lake.md)  
Next: [Prompt/Response Pairs for Model Tasks](./06-prompt-response-pairs.md)
