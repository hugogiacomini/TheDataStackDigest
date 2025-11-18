# Model Task Selection for Business Requirements

## Overview

Match business objectives to appropriate model tasks: text generation, classification, summarization, extraction, translation, code generation, or embedding.

## Common Model Tasks

| Task | Business Use Case | Example Models |
|------|-------------------|----------------|
| **Text Generation** | Content creation, chatbots, creative writing | GPT-4, Llama 3.1, Mistral |
| **Classification** | Sentiment analysis, intent detection, content moderation | BERT, RoBERTa, DistilBERT |
| **Summarization** | Document summarization, meeting notes | BART, T5, Llama 3.1 |
| **Named Entity Recognition (NER)** | Extract entities (names, dates, locations) | spaCy models, BERT-NER |
| **Question Answering** | Customer support, knowledge retrieval | RAG systems with Llama/GPT |
| **Translation** | Multilingual support | mT5, NLLB, GPT-4 |
| **Code Generation** | Developer assistance, automation | Codex, StarCoder, Code Llama |
| **Embedding** | Semantic search, clustering, recommendation | BGE, E5, OpenAI embeddings |

## Hands-on Examples

### Business Requirement → Task Mapping

```python
# Requirement: "Categorize customer emails into support, sales, billing"
# Task: Multi-class text classification

from transformers import pipeline

classifier = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")
result = classifier("I need help resetting my password")
# Output: {'label': 'support', 'score': 0.95}
```

### Databricks Foundation Model API for Summarization

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import ChatMessage, ChatMessageRole

w = WorkspaceClient()

messages = [
    ChatMessage(
        role=ChatMessageRole.SYSTEM,
        content="You are a summarization assistant. Provide concise summaries."
    ),
    ChatMessage(
        role=ChatMessageRole.USER,
        content="Summarize this 500-word article about renewable energy trends in 3 bullet points."
    )
]

response = w.serving_endpoints.query(
    name="databricks-meta-llama-3-1-405b-instruct",
    messages=messages,
    max_tokens=150
)
```

### Code Generation for Automation

```python
# Requirement: "Generate SQL queries from natural language"
# Task: Code generation

prompt = """
Convert this request to SQL:
"Find all customers in California who made purchases over $1000 in the last 30 days"

Table schema: customers (id, name, state), orders (customer_id, amount, order_date)
"""

# LLM generates:
# SELECT c.id, c.name FROM customers c
# JOIN orders o ON c.id = o.customer_id
# WHERE c.state = 'CA' AND o.amount > 1000
# AND o.order_date >= CURRENT_DATE - INTERVAL 30 DAY
```

## Task Selection Criteria

1. **Input/Output Type**: Text-to-text, text-to-label, text-to-embedding.
2. **Latency Requirements**: Real-time (smaller models) vs batch (larger models).
3. **Accuracy Needs**: High-stakes (GPT-4) vs casual (Llama 3 8B).
4. **Cost**: API calls vs self-hosted; larger models = higher cost.
5. **Domain Specificity**: General-purpose vs fine-tuned (medical, legal, code).

## Best Practices

- Start with general-purpose models; fine-tune if domain-specific accuracy needed.
- For classification, consider if zero-shot prompting suffices before training.
- Use smaller models for latency-sensitive apps; reserve large models for complex reasoning.
- Validate task choice with pilot evaluation before full deployment.

## Sample Questions

1. Which task for categorizing support tickets?
2. Best model type for semantic search?
3. When use summarization vs extraction?
4. How choose between GPT-4 and Llama 3 70B?
5. Task for generating Python code from comments?

## Answers

1. Multi-class text classification.
2. Embedding models (BGE, E5) for vector similarity.
3. Summarization condenses; extraction pulls specific facts verbatim.
4. GPT-4 for complex reasoning/accuracy; Llama 3 70B for cost-efficiency and control.
5. Code generation task.

## References

- [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html)
- [Hugging Face Tasks](https://huggingface.co/tasks)
- [LangChain Use Cases](https://python.langchain.com/docs/use_cases/)

---

Previous: [Prompt Engineering](./01-prompt-engineering-formatted-responses.md)  
Next: [Chain Component Selection](./03-chain-component-selection.md)
