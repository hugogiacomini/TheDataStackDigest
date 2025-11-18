# Utilizing Databricks Agent Framework for Agentic Systems

## Overview

Build multi-step reasoning agents using Databricks Agent Framework: tool integration, agent orchestration, state management, and deployment.

## Agent Architecture

```text
User Query → Agent (LLM) → Tool Selection → Tool Execution → Response Synthesis
             ↑                                     ↓
             └─────────── Observation ─────────────┘
```

**Agent Components**:

- **LLM**: Decision-making engine
- **Tools**: Functions agent can invoke
- **Memory**: Conversation/state tracking
- **Orchestrator**: Manages reasoning loop

## Hands-on Examples

### Basic Agent with Tools

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain.tools import Tool
from langchain_community.chat_models import ChatDatabricks
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder

# Define tools
def search_knowledge_base(query: str) -> str:
    """Search internal documentation."""
    # Simplified: would use vector search
    return f"Documentation result for: {query}"

def query_database(sql_query: str) -> str:
    """Execute SQL query on database."""
    # Simplified: would use Databricks SQL
    return f"Query result: 42 records found"

def send_email(recipient: str, message: str) -> str:
    """Send email to user."""
    return f"Email sent to {recipient}"

tools = [
    Tool(
        name="search_docs",
        func=search_knowledge_base,
        description="Search internal documentation and knowledge base. Use for questions about product features, setup, and best practices."
    ),
    Tool(
        name="query_db",
        func=query_database,
        description="Execute SQL queries on the database. Use for questions about data, orders, customers, or statistics."
    ),
    Tool(
        name="send_email",
        func=send_email,
        description="Send email to a user. Use when you need to notify someone or escalate an issue."
    )
]

# Create agent
llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")

prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a helpful customer support agent.
    
Use available tools to answer questions and help users.
Think step-by-step about which tools to use.
If you need information from multiple sources, call tools sequentially."""),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad")
])

agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# Execute agent
response = agent_executor.invoke({
    "input": "How many orders did customer john@example.com place last month?"
})
print(response["output"])
```

### ReAct Agent Pattern

```python
from langchain.agents import create_react_agent

react_prompt = ChatPromptTemplate.from_template("""Answer the following question using this format:

Question: {input}

Thought: Think about what information you need
Action: The tool to use
Action Input: The input to the tool
Observation: The result from the tool

... (repeat Thought/Action/Observation as needed)

Thought: I now know the final answer
Final Answer: The complete answer to the question

Available tools: {tool_names}
Tool descriptions: {tools}

Question: {input}
{agent_scratchpad}
""")

react_agent = create_react_agent(llm, tools, react_prompt)
react_executor = AgentExecutor(agent=react_agent, tools=tools, verbose=True)
```

### Databricks Agent Framework (MLflow)

```python
import mlflow
from databricks.agents import Agent, Tool as DBTool
from databricks.agents.tools import SearchTool, PythonREPLTool

# Define agent with Databricks framework
class CustomerSupportAgent(Agent):
    def __init__(self):
        super().__init__(
            name="customer_support_agent",
            llm_endpoint="databricks-meta-llama-3-1-70b-instruct",
            tools=[
                SearchTool(
                    index_name="main.rag_demo.docs_vector_index",
                    description="Search product documentation"
                ),
                DBTool(
                    name="check_order_status",
                    function=self.check_order_status,
                    description="Check order status by order ID"
                )
            ],
            system_prompt="""You are a customer support agent.
            Help users with their questions using available tools.
            Be concise and helpful."""
        )
    
    def check_order_status(self, order_id: str) -> dict:
        """Query order status from database."""
        # Simplified implementation
        return {
            "order_id": order_id,
            "status": "shipped",
            "tracking": "TRACK123"
        }

# Log agent to MLflow
with mlflow.start_run():
    agent = CustomerSupportAgent()
    mlflow.agents.log_agent(agent, "customer_support_agent")
    
    # Register to Unity Catalog
    mlflow.register_model(
        f"runs:/{mlflow.active_run().info.run_id}/customer_support_agent",
        "main.ml_models.support_agent"
    )
```

### Multi-Agent System

```python
from typing import List, Dict

class MultiAgentOrchestrator:
    def __init__(self):
        self.agents = {
            "triage": self.create_triage_agent(),
            "technical": self.create_technical_agent(),
            "billing": self.create_billing_agent()
        }
    
    def create_triage_agent(self) -> AgentExecutor:
        """Agent that routes to specialist agents."""
        
        def route_to_technical(query: str) -> str:
            return "ROUTE_TECHNICAL"
        
        def route_to_billing(query: str) -> str:
            return "ROUTE_BILLING"
        
        tools = [
            Tool(name="route_technical", func=route_to_technical, 
                 description="Route to technical support for product issues"),
            Tool(name="route_billing", func=route_to_billing,
                 description="Route to billing for payment/invoice questions")
        ]
        
        return create_tool_calling_agent(llm, tools, prompt)
    
    def create_technical_agent(self) -> AgentExecutor:
        """Technical support specialist."""
        # ... technical tools
        pass
    
    def create_billing_agent(self) -> AgentExecutor:
        """Billing specialist."""
        # ... billing tools
        pass
    
    def process_query(self, query: str) -> str:
        """Route query through agents."""
        
        # Step 1: Triage
        triage_result = self.agents["triage"].invoke({"input": query})
        
        # Step 2: Route to specialist
        if "TECHNICAL" in triage_result["output"]:
            return self.agents["technical"].invoke({"input": query})["output"]
        elif "BILLING" in triage_result["output"]:
            return self.agents["billing"].invoke({"input": query})["output"]
        else:
            return triage_result["output"]
```

### Agent with Memory

```python
from langchain.memory import ConversationBufferMemory

# Create memory
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# Create agent with memory
agent_with_memory = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)

# Multi-turn conversation
agent_with_memory.invoke({"input": "What is Unity Catalog?"})
agent_with_memory.invoke({"input": "How do I set it up?"})  # Context from previous turn
```

### Agent Evaluation and Monitoring

```python
import pandas as pd

class AgentEvaluator:
    def __init__(self, agent: AgentExecutor):
        self.agent = agent
    
    def evaluate(self, test_cases: List[Dict]) -> pd.DataFrame:
        """Evaluate agent on test cases."""
        
        results = []
        for case in test_cases:
            try:
                response = self.agent.invoke({"input": case["query"]})
                
                # Check if correct tools used
                tools_used = self.extract_tools_from_response(response)
                correct_tools = set(tools_used) == set(case["expected_tools"])
                
                results.append({
                    "query": case["query"],
                    "response": response["output"],
                    "tools_used": tools_used,
                    "correct_tools": correct_tools,
                    "success": True
                })
            except Exception as e:
                results.append({
                    "query": case["query"],
                    "response": None,
                    "tools_used": [],
                    "correct_tools": False,
                    "success": False,
                    "error": str(e)
                })
        
        return pd.DataFrame(results)
    
    def extract_tools_from_response(self, response: dict) -> List[str]:
        # Extract tools from agent scratchpad
        return []

# Usage
test_cases = [
    {
        "query": "How many orders for john@example.com?",
        "expected_tools": ["query_db"]
    },
    {
        "query": "What is Delta Lake and how many users do we have?",
        "expected_tools": ["search_docs", "query_db"]
    }
]

evaluator = AgentEvaluator(agent_executor)
eval_results = evaluator.evaluate(test_cases)
print(eval_results)
```

## Best Practices

- **Clear tool descriptions**: LLM relies on descriptions to select tools; be specific about when to use each.
- **Error handling**: Tools should return informative errors, not exceptions.
- **Tool composition**: Design tools to be composable; agent can chain them.
- **Limit iterations**: Set max iterations to prevent infinite loops.
- **Logging**: Track tool calls, reasoning steps for debugging.
- **Evaluation**: Test agent on diverse queries before production.

## Sample Questions

1. What is the ReAct pattern?
2. How does agent select tools?
3. When use multi-agent system?
4. How add memory to agent?
5. Best practice for tool design?

## Answers

1. Reasoning + Acting pattern; agent alternates between thinking (reasoning) and action (tool use) until reaching answer.
2. LLM reads tool descriptions and chooses based on query requirements; relies on clear, specific descriptions.
3. When different query types need different expertise (technical vs billing), when workflow has distinct stages, or when scaling single agent becomes complex.
4. Add ConversationBufferMemory to AgentExecutor; stores chat history for context across turns.
5. Single responsibility per tool, descriptive names and docs, return structured data, handle errors gracefully, idempotent when possible.

## References

- [Databricks Agent Framework](https://docs.databricks.com/en/generative-ai/agent-framework/index.html)
- [LangChain Agents](https://python.langchain.com/docs/modules/agents/)

---

Previous: [Model Selection Based on Requirements](./06-model-selection-requirements.md)  
Next: [Implementing LLM Guardrails](./01-implementing-llm-guardrails.md)
