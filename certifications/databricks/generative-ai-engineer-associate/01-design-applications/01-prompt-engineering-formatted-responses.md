# Prompt Engineering for Formatted Responses

## Overview

Design prompts that elicit specifically structured outputs (JSON, tables, lists) from LLMs by providing clear format instructions, examples, and constraints.

## Key Concepts

**Structured Output Formats**: JSON, YAML, CSV, Markdown tables, numbered lists, XML.

**Techniques**:

- **Explicit format specification**: State the exact output structure.
- **Few-shot examples**: Show 2-3 examples of desired format.
- **Delimiters and markers**: Use XML tags, JSON keys, or special tokens.
- **Constraints**: Specify field types, required keys, value ranges.

## Hands-on Examples

### JSON Output

```python
prompt = """
Extract the following information from the text and return as JSON with keys: name, age, occupation.

Text: "Sarah Johnson, 34 years old, works as a data scientist."

Output format:
{
  "name": "string",
  "age": integer,
  "occupation": "string"
}
"""

# LLM response: {"name": "Sarah Johnson", "age": 34, "occupation": "data scientist"}
```

### Markdown Table

```python
prompt = """
Summarize the quarterly sales data in a Markdown table with columns: Quarter, Revenue, Growth%.

Data: Q1 $1.2M, Q2 $1.5M (+25%), Q3 $1.8M (+20%), Q4 $2.1M (+16.7%)

| Quarter | Revenue | Growth% |
|---------|---------|---------|
"""
```

### Structured List with Databricks SDK

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import ChatMessage, ChatMessageRole

w = WorkspaceClient()

messages = [
    ChatMessage(
        role=ChatMessageRole.USER,
        content="""List 3 key benefits of RAG systems. 
Format as:
1. [Benefit]: [Description]
2. [Benefit]: [Description]
3. [Benefit]: [Description]"""
    )
]

response = w.serving_endpoints.query(
    name="databricks-meta-llama-3-1-70b-instruct",
    messages=messages
)
```

## Best Practices

- Provide output schema upfront; use Pydantic models for validation.
- Use system prompts to enforce format globally.
- Add validation: "If uncertain, return null for that field."
- For complex nested structures, break into multi-step prompts.
- Test with edge cases (missing data, malformed input).

## Sample Questions

1. How elicit JSON output from an LLM?
2. Why use few-shot examples for formatting?
3. Which delimiter helps parse structured responses?
4. How enforce required fields in output?
5. When use system vs user prompts for format?

## Answers

1. Specify JSON schema in prompt with example; optionally use JSON mode if supported.
2. Shows model exact format; reduces ambiguity and improves consistency.
3. XML tags, triple backticks, or JSON keys clearly separate content.
4. State "Required fields: [list]" and add validation in post-processing.
5. System prompt for global format rules; user prompt for instance-specific structure.

## References

- [Databricks Foundation Model APIs](https://docs.databricks.com/en/machine-learning/foundation-models/index.html)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)
- [LangChain Output Parsers](https://python.langchain.com/docs/modules/model_io/output_parsers/)

---

Next: [Model Task Selection](./02-model-task-selection.md)
