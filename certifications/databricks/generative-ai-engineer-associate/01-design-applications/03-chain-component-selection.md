# Chain Component Selection for Inputs and Outputs

## Overview

Select and configure chain components (prompts, retrievers, models, output parsers, memory) to transform desired inputs into target outputs.

## Core Chain Components

1. **Prompt Templates**: Format user input with context and instructions.
2. **Retrievers**: Fetch relevant documents from vector stores or databases.
3. **Language Models**: Generate or transform text (LLM, chat model, embedding model).
4. **Output Parsers**: Structure model output into usable formats (JSON, lists).
5. **Memory**: Maintain conversation history or state across interactions.
6. **Runnables**: Compose components into pipelines with LangChain Expression Language (LCEL).

## Hands-on Examples

### Simple Chain: Question Answering

```python
from langchain_community.chat_models import ChatDatabricks
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Components
llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
prompt = ChatPromptTemplate.from_template("Answer this question concisely: {question}")
parser = StrOutputParser()

# Chain
chain = prompt | llm | parser

# Execute
result = chain.invoke({"question": "What is RAG?"})
print(result)  # "RAG stands for Retrieval-Augmented Generation..."
```

### RAG Chain with Retriever

```python
from langchain_community.vectorstores import DatabricksVectorSearch
from langchain_community.embeddings import DatabricksEmbeddings
from langchain_core.runnables import RunnablePassthrough

# Components
embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")
vectorstore = DatabricksVectorSearch(
    index_name="main.rag_demo.docs_index",
    embedding=embeddings
)
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

rag_prompt = ChatPromptTemplate.from_template("""
Answer the question using only the provided context.

Context: {context}

Question: {question}
""")

# Chain with retriever
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | rag_prompt
    | llm
    | parser
)

result = rag_chain.invoke("What are the benefits of Delta Lake?")
```

### Chain with Output Parser

```python
from langchain_core.output_parsers import JsonOutputParser
from pydantic import BaseModel, Field

class ProductExtraction(BaseModel):
    name: str = Field(description="Product name")
    price: float = Field(description="Price in USD")
    category: str = Field(description="Product category")

json_parser = JsonOutputParser(pydantic_object=ProductExtraction)

extraction_prompt = ChatPromptTemplate.from_template("""
Extract product information from the text and return as JSON.

Text: {text}

{format_instructions}
""")

extraction_chain = (
    extraction_prompt.partial(format_instructions=json_parser.get_format_instructions())
    | llm
    | json_parser
)

result = extraction_chain.invoke({"text": "The UltraBook Pro costs $1299 and is in the laptop category."})
# Output: {'name': 'UltraBook Pro', 'price': 1299.0, 'category': 'laptop'}
```

## Component Selection Criteria

| Desired I/O | Key Components | Example |
|-------------|----------------|---------|
| Text → Structured Data | Prompt + LLM + JSON Parser | Extract entities from documents |
| Question → Contextual Answer | Retriever + Prompt + LLM | RAG knowledge base |
| Conversation → Response | Memory + Prompt + Chat Model | Multi-turn chatbot |
| Document → Summary | Prompt + LLM + String Parser | Summarization pipeline |

## Best Practices

- Use LCEL (pipe operator `|`) for composable, inspectable chains.
- Add output parsers early to enforce structure and reduce post-processing.
- Include retrievers only when external knowledge is required.
- For multi-turn conversations, add `ConversationBufferMemory` or session state.
- Test each component independently before chaining.

## Sample Questions

1. Which component fetches external documents?
2. Role of output parser in a chain?
3. When add memory to a chain?
4. How compose multiple components in LangChain?
5. Difference between prompt template and retriever?

## Answers

1. Retriever (e.g., vector store retriever, SQL retriever).
2. Structures LLM output into usable formats (JSON, lists, objects).
3. When maintaining conversation history or stateful interactions.
4. Use LCEL with pipe operator: `component1 | component2 | component3`.
5. Prompt formats inputs; retriever fetches relevant data from external sources.

## References

- [LangChain Chains](https://python.langchain.com/docs/modules/chains/)
- [LangChain Expression Language](https://python.langchain.com/docs/expression_language/)
- [Databricks LangChain Integration](https://docs.databricks.com/en/generative-ai/langchain.html)

---

Previous: [Model Task Selection](./02-model-task-selection.md)  
Next: [Translating Business Goals to AI Pipeline Specs](./04-business-goals-to-pipeline-specs.md)
