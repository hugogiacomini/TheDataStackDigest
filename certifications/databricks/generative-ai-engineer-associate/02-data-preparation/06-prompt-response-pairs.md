# Identifying Prompt/Response Pairs for Model Tasks

## Overview

Curate high-quality prompt/response pairs aligned with specific model tasks (instruction-tuning, fine-tuning, evaluation datasets).

## Pair Requirements by Task

| Task | Prompt Characteristics | Response Characteristics |
|------|----------------------|------------------------|
| Classification | Clear input text | Single label or scores |
| Summarization | Long document | Concise summary |
| QA | Specific question | Factual answer with citations |
| Instruction-following | Task description | Correct execution |
| Code generation | Natural language request | Syntactically correct code |

## Hands-on Examples

### QA Pair Curation

```python
qa_pairs = [
    {
        "prompt": "What is the default shuffle partition count in Spark?",
        "response": "200 partitions (controlled by spark.sql.shuffle.partitions configuration).",
        "metadata": {"domain": "spark", "difficulty": "easy"}
    },
    {
        "prompt": "How does Delta Lake ensure ACID compliance?",
        "response": "Delta Lake uses a transaction log that records all changes atomically. It implements optimistic concurrency control and versioning to ensure Atomicity, Consistency, Isolation, and Durability.",
        "metadata": {"domain": "delta", "difficulty": "medium"}
    }
]
```

### Classification Pairs

```python
classification_pairs = [
    {
        "prompt": "Classify the sentiment: 'The product exceeded my expectations!'",
        "response": "positive",
        "metadata": {"task": "sentiment", "confidence": 0.95}
    }
]
```

### Instruction-Tuning Pairs

```python
instruction_pairs = [
    {
        "prompt": "Convert this JSON to a Python dictionary and extract the 'name' field:\n{\"user\": {\"name\": \"Alice\", \"age\": 30}}",
        "response": "```python\nimport json\ndata = json.loads('{\"user\": {\"name\": \"Alice\", \"age\": 30}}')\nname = data['user']['name']\nprint(name)  # Output: Alice\n```",
        "metadata": {"task": "code_generation", "language": "python"}
    }
]
```

## Data Collection Strategies

1. **Historical logs**: Mine support tickets, chat transcripts.
2. **Expert annotation**: SMEs create gold-standard pairs.
3. **Synthetic generation**: Use strong LLM to generate diverse pairs.
4. **User feedback**: Collect thumbs up/down on production responses.

## Quality Checks

```python
def validate_pair(prompt, response):
    """Check if prompt/response pair meets quality criteria."""
    
    issues = []
    
    # Check lengths
    if len(prompt.split()) < 3:
        issues.append("Prompt too short")
    if len(response.split()) < 5:
        issues.append("Response too short")
    
    # Check for placeholder text
    if any(placeholder in response.lower() for placeholder in ['lorem ipsum', '[insert', 'todo']):
        issues.append("Contains placeholder text")
    
    # Check prompt clarity
    if '?' not in prompt and not prompt.endswith(':'):
        issues.append("Prompt lacks clear question or instruction")
    
    return len(issues) == 0, issues
```

## Sample Questions

1. What makes a good QA pair?
2. How ensure response quality?
3. Difference between prompt for fine-tuning vs inference?
4. When use synthetic data generation?
5. How balance dataset difficulty?

## Answers

1. Clear, specific question; factual, complete answer; verifiable against source.
2. Human review, automated checks (length, format), inter-annotator agreement, pilot testing.
3. Fine-tuning prompts are training examples (more diverse, cover edge cases); inference prompts are user-facing (specific to task).
4. When insufficient real data; to augment diversity; to test model robustness.
5. Include easy (baseline), medium (typical), hard (edge cases) examples; stratify by difficulty in splits.

## References

- [Instruction Tuning](https://arxiv.org/abs/2109.01652)
- [Data Curation for LLMs](https://huggingface.co/blog/instruction-tuning)

---

Previous: [Source Document Selection](./05-source-document-selection.md)  
Next: [Retrieval Performance Evaluation](./07-retrieval-performance-evaluation.md)
