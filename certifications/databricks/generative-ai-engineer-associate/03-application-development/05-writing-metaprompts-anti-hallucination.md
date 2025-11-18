# Writing Metaprompts to Minimize Hallucinations and Data Leakage

## Overview

Design metaprompts (system-level instructions) that constrain LLM behavior to prevent hallucinations, data leakage, and off-topic responses in production RAG applications.

## Metaprompt Components

| Component | Purpose | Example |
|-----------|---------|---------|
| **Role definition** | Set assistant identity | "You are a customer support agent for Acme Corp" |
| **Scope limitation** | Define knowledge boundaries | "Only answer using provided context" |
| **Hallucination prevention** | Explicit constraints | "If unsure, say 'I don't know'" |
| **Data protection** | Prevent leakage | "Never reveal internal system details" |
| **Output formatting** | Consistency | "Always cite sources with [1], [2]" |

## Hands-on Examples

### Anti-Hallucination Metaprompt

```python
ANTI_HALLUCINATION_METAPROMPT = """You are a technical support assistant for Databricks products.

CRITICAL RULES:
1. Answer ONLY using information from the provided CONTEXT
2. If the CONTEXT does not contain the answer, respond: "I don't have enough information to answer this question. Please check the documentation or contact support."
3. Do NOT use your pre-training knowledge
4. Do NOT make up information, URLs, or feature names
5. If only partial information is available, state what you know and what is missing

ANSWER FORMAT:
- Start with a direct answer
- Provide supporting details from context
- Cite sources with [1], [2], etc.
- End with relevant links if provided in context

CONTEXT:
{context}

QUESTION:
{question}

ANSWER:"""

# Usage
from langchain_community.chat_models import ChatDatabricks

llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")

def safe_rag_query(question: str, context: str) -> str:
    prompt = ANTI_HALLUCINATION_METAPROMPT.format(
        context=context,
        question=question
    )
    return llm.invoke([HumanMessage(content=prompt)]).content
```

### Data Leakage Prevention Metaprompt

```python
DATA_PROTECTION_METAPROMPT = """You are an internal knowledge assistant.

DATA PROTECTION RULES:
1. NEVER reveal:
   - System architecture details
   - Internal URLs or endpoints
   - API keys, credentials, or tokens
   - Employee names or contact information
   - Unreleased product features
   - Internal code or configurations

2. If asked about protected information:
   - Respond: "I cannot provide that information due to security policies."
   - Do NOT explain why the information exists or confirm its existence

3. Safe to share:
   - Public documentation
   - General product features (already released)
   - Publicly available tutorials

CONTEXT:
{context}

QUESTION:
{question}

ANSWER:"""

def protected_query(question: str, context: str) -> str:
    """Query with data protection guardrails."""
    prompt = DATA_PROTECTION_METAPROMPT.format(
        context=context,
        question=question
    )
    return llm.invoke([HumanMessage(content=prompt)]).content
```

### Comprehensive RAG Metaprompt Template

```python
COMPREHENSIVE_RAG_METAPROMPT = """You are {role_description}.

IDENTITY & SCOPE:
{scope_definition}

ANSWER REQUIREMENTS:
1. Grounding: Use ONLY the provided CONTEXT below. Do not use pre-training knowledge.
2. Accuracy: If the CONTEXT lacks sufficient information, say: "{insufficient_info_response}"
3. Citations: Reference information with [1], [2] corresponding to source documents.
4. Clarity: Use clear, professional language appropriate for {audience}.
5. Completeness: Address all parts of the question if information is available.

PROHIBITED ACTIONS:
{prohibited_actions}

OUTPUT FORMAT:
{output_format}

---

CONTEXT:
{context}

---

USER QUESTION:
{question}

---

YOUR ANSWER:"""

# Example configuration
rag_config = {
    "role_description": "a technical documentation assistant for Databricks Unity Catalog",
    "scope_definition": "Answer questions about Unity Catalog features, setup, and best practices using official documentation",
    "insufficient_info_response": "I don't have sufficient information in the documentation to answer this question accurately. Please refer to docs.databricks.com or contact support.",
    "audience": "data engineers and administrators",
    "prohibited_actions": """
    - DO NOT speculate or infer beyond the provided context
    - DO NOT provide information about unreleased features
    - DO NOT share internal system details
    - DO NOT make up examples not present in context
    """,
    "output_format": """
    - Start with a direct answer (1-2 sentences)
    - Provide detailed explanation with steps if applicable
    - Include code examples if present in context
    - End with citations: [1] source_document.pdf
    """
}

def create_rag_prompt(question: str, context: str, config: dict) -> str:
    return COMPREHENSIVE_RAG_METAPROMPT.format(
        **config,
        context=context,
        question=question
    )
```

### Domain-Specific Constraints

```python
MEDICAL_RAG_METAPROMPT = """You are a medical information assistant.

CRITICAL SAFETY RULES:
1. ONLY provide information from peer-reviewed sources in the CONTEXT
2. ALWAYS include disclaimer: "This is for informational purposes only. Consult a healthcare provider for medical advice."
3. NEVER diagnose conditions or recommend specific treatments
4. If question asks for diagnosis or treatment recommendation, respond: "I cannot provide medical diagnoses or treatment recommendations. Please consult a qualified healthcare provider."

CONTEXT:
{context}

QUESTION:
{question}

ANSWER:"""

FINANCIAL_RAG_METAPROMPT = """You are a financial information assistant.

COMPLIANCE RULES:
1. Provide ONLY factual, educational information from CONTEXT
2. NEVER give investment advice or recommendations
3. ALWAYS include: "This is not financial advice. Consult a financial advisor."
4. If asked for advice, respond: "I provide educational information only, not financial advice."

CONTEXT:
{context}

QUESTION:
{question}

ANSWER:"""
```

### Multi-Turn Conversation Metaprompt

```python
CONVERSATIONAL_METAPROMPT = """You are a helpful assistant with conversation memory.

CONVERSATION RULES:
1. Reference previous messages when relevant
2. If a follow-up question lacks context, ask for clarification
3. Stay consistent with previous answers
4. Correct yourself if new information contradicts previous statements

MEMORY:
{conversation_history}

CURRENT CONTEXT:
{context}

CURRENT QUESTION:
{question}

RESPONSE:"""

class ConversationalRAG:
    def __init__(self):
        self.history = []
        self.llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
    
    def query(self, question: str, context: str) -> str:
        # Format conversation history
        history_str = "\n".join([
            f"User: {h['question']}\nAssistant: {h['answer']}"
            for h in self.history[-3:]  # Keep last 3 turns
        ])
        
        prompt = CONVERSATIONAL_METAPROMPT.format(
            conversation_history=history_str if history_str else "No previous conversation",
            context=context,
            question=question
        )
        
        answer = self.llm.invoke([HumanMessage(content=prompt)]).content
        
        # Store in history
        self.history.append({"question": question, "answer": answer})
        
        return answer
```

### Self-Correction Metaprompt

```python
SELF_CORRECTION_METAPROMPT = """You are an assistant that verifies its own answers.

STEP 1 - INITIAL ANSWER:
Based on the CONTEXT below, answer the QUESTION.

CONTEXT:
{context}

QUESTION:
{question}

INITIAL ANSWER:
{initial_answer}

STEP 2 - VERIFICATION:
Now verify your answer:
1. Is every claim supported by the CONTEXT?
2. Did you add information not in the CONTEXT?
3. Are citations accurate?

If issues found, provide CORRECTED ANSWER.
If no issues, respond: "VERIFIED - Answer is accurate."

VERIFICATION RESULT:"""

def self_correcting_rag(question: str, context: str) -> dict:
    """RAG with self-verification."""
    
    # Initial answer
    initial_prompt = f"""Using only this context: {context}

Answer: {question}"""
    
    initial_answer = llm.invoke([HumanMessage(content=initial_prompt)]).content
    
    # Verification
    verification_prompt = SELF_CORRECTION_METAPROMPT.format(
        context=context,
        question=question,
        initial_answer=initial_answer
    )
    
    verification = llm.invoke([HumanMessage(content=verification_prompt)]).content
    
    return {
        "initial_answer": initial_answer,
        "verification": verification,
        "needs_correction": "VERIFIED" not in verification
    }
```

## Best Practices

- **Explicit constraints**: Be direct about what model should/shouldn't do.
- **Clear fallback**: Define exact response for insufficient information.
- **Citation requirement**: Force grounding by requiring source references.
- **Domain-specific**: Tailor metaprompt to use case (medical, financial, etc.).
- **Test edge cases**: Verify metaprompt prevents common failure modes.
- **Version control**: Track metaprompt changes alongside model versions.

## Sample Questions

1. What is a metaprompt?
2. How prevent hallucinations in metaprompts?
3. Key components of data protection metaprompt?
4. When use self-correction approach?
5. Trade-off of strict metaprompts?

## Answers

1. System-level prompt that sets constraints, role, and rules for all subsequent queries.
2. Explicit "only use context" instruction, require citations, define fallback response for unknown, prohibit speculation.
3. List prohibited information types, define response for protected questions, specify safe information boundaries, never confirm/deny protected data existence.
4. For high-stakes applications (medical, legal, financial), when accuracy critical, or when model prone to confident errors.
5. May refuse valid queries (false negatives), reduce model flexibility, increase latency for multi-step verification; balance with risk tolerance.

## References

- [Constitutional AI](https://www.anthropic.com/index/constitutional-ai-harmlessness-from-ai-feedback)
- [Prompt Injection Defense](https://learnprompting.org/docs/prompt_hacking/defensive_measures/overview)

---

Previous: [Prompt Formatting Effects](./04-prompt-formatting-effects.md)  
Next: [Model Selection Based on Requirements](./06-model-selection-requirements.md)
