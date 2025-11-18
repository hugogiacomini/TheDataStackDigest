# Multi-Stage Reasoning with Tools and Agents

## Overview

Define and sequence tools (functions, APIs, databases) that agents use to gather knowledge or take actions across multiple reasoning stages.

## Key Concepts

**Tools**: Functions an agent can invoke to interact with external systems (search, calculate, query DB, call API).

**Multi-Stage Reasoning**: Agent breaks complex task into steps, selecting appropriate tools at each stage.

**Agent Frameworks**: LangChain Agents, ReAct, LlamaIndex Agents, Databricks Agent Framework.

## Tool Definition

```python
from langchain.tools import Tool
from langchain.agents import initialize_agent, AgentType
from langchain_community.chat_models import ChatDatabricks

def search_documentation(query: str) -> str:
    """Search internal documentation for relevant information."""
    # Simulate vector search
    return f"Found 3 relevant docs about {query}"

def query_database(sql: str) -> str:
    """Execute SQL query against data warehouse."""
    # Simulate DB query
    return f"Query result: 42 rows returned"

def send_email(recipient: str, subject: str, body: str) -> str:
    """Send email notification."""
    return f"Email sent to {recipient}"

# Define tools
tools = [
    Tool(
        name="SearchDocs",
        func=search_documentation,
        description="Search internal documentation. Input: query string. Returns: relevant doc snippets."
    ),
    Tool(
        name="QueryDB",
        func=query_database,
        description="Execute SQL query. Input: SQL statement. Returns: query results."
    ),
    Tool(
        name="SendEmail",
        func=send_email,
        description="Send email. Input: recipient, subject, body (comma-separated). Returns: confirmation."
    )
]
```

## Multi-Stage Reasoning Example

### Use Case: Customer Order Investigation

```python
from langchain.agents import initialize_agent, AgentType
from langchain_community.chat_models import ChatDatabricks

llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# Complex query requiring multiple tools
query = """
Customer John Doe (customer_id: 12345) is asking about delayed order #ORD-789. 
Check order status in database, search docs for shipping policy, and if delayed >3 days, send apology email.
"""

result = agent.run(query)

# Agent reasoning trace:
# 1. Thought: Need to check order status first
#    Action: QueryDB
#    Input: SELECT status, ship_date FROM orders WHERE order_id = 'ORD-789'
#    Observation: Order shipped 5 days ago, status='in_transit'
#
# 2. Thought: Delayed >3 days, need to check policy
#    Action: SearchDocs
#    Input: shipping delay policy
#    Observation: Policy states apology email for delays >3 days
#
# 3. Thought: Should send apology email
#    Action: SendEmail
#    Input: john.doe@example.com, Apology for Delayed Order, We sincerely apologize...
#    Observation: Email sent to john.doe@example.com
#
# Final Answer: Order ORD-789 is delayed by 5 days. Apology email sent per company policy.
```

## Tool Sequencing Strategies

### Sequential (Chain of Thought)

```python
# Order matters: retrieve → analyze → act
tool_sequence = [
    "SearchDocs",      # 1. Gather knowledge
    "QueryDB",         # 2. Get current state
    "SendEmail"        # 3. Take action
]
```

### Conditional (Branching)

```python
def conditional_routing(query):
    if "urgent" in query.lower():
        return ["QueryDB", "SendEmail"]  # Skip docs, act fast
    else:
        return ["SearchDocs", "QueryDB", "SendEmail"]  # Full workflow
```

### Parallel (When Independent)

```python
from langchain.schema.runnable import RunnableParallel

# Execute tools concurrently
parallel_tools = RunnableParallel(
    docs=search_documentation,
    db=query_database
)
```

## Databricks Agent Framework

```python
from databricks import agents
from databricks.sdk import WorkspaceClient

w = WorkspaceClient()

# Define agent with tools
agent_config = {
    "name": "customer_support_agent",
    "instructions": "You help investigate customer issues using available tools.",
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "search_docs",
                "description": "Search documentation",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string"}
                    },
                    "required": ["query"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "query_db",
                "description": "Query database",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "sql": {"type": "string"}
                    },
                    "required": ["sql"]
                }
            }
        }
    ]
}

# Deploy agent
# agents.deploy(agent_config, model_name="agent_v1")
```

## Best Practices

- Define clear tool descriptions; agent relies on these for selection.
- Limit tool count (<10); too many confuses the agent.
- Add error handling in tools (timeouts, invalid inputs).
- Log tool calls for debugging and auditing.
- Use structured output from tools (JSON) for easier parsing.
- Order tools by frequency of use in descriptions.

## Sample Questions

1. What is a tool in agent context?
2. How does agent choose which tool to use?
3. When use sequential vs parallel tool execution?
4. Role of tool description?
5. How debug agent tool selection errors?

## Answers

1. Function/API agent invokes to gather data or take actions.
2. Based on tool descriptions matching task requirements in LLM reasoning.
3. Sequential when tools depend on each other; parallel when independent.
4. Critical for agent to understand when/how to use the tool; poor descriptions lead to misuse.
5. Enable verbose mode; review tool call logs; refine tool descriptions or add examples.

## References

- [LangChain Agents](https://python.langchain.com/docs/modules/agents/)
- [Databricks Agent Framework](https://docs.databricks.com/en/generative-ai/agent-framework/index.html)
- [ReAct: Reasoning and Acting](https://arxiv.org/abs/2210.03629)

---

Previous: [Business Goals to Pipeline Specs](./04-business-goals-to-pipeline-specs.md)  
Next: [Chunking Strategies](../02-data-preparation/01-chunking-strategies.md)
