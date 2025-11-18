# Creating Data Retrieval Tools for Specific Needs

## Overview

Build custom retrieval tools that fetch data from various sources (vector stores, databases, APIs, documents) to augment LLM context for agent-based applications.

## Tool Anatomy

A retrieval tool consists of:

- **Name**: Descriptive identifier
- **Description**: What it retrieves (used by LLM for selection)
- **Parameters**: Input schema
- **Function**: Retrieval logic

## Hands-on Examples

### Basic Vector Store Retrieval Tool

```python
from langchain.tools import Tool
from langchain_community.vectorstores import DatabricksVectorSearch
from langchain_community.embeddings import DatabricksEmbeddings

def create_vector_search_tool(index_name: str, description: str) -> Tool:
    """Create tool for semantic search over vector index."""
    
    embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")
    vectorstore = DatabricksVectorSearch(
        endpoint="rag_vector_search_endpoint",
        index_name=index_name,
        embedding=embeddings,
        text_column="content"
    )
    
    def search_documents(query: str) -> str:
        """Retrieve relevant documents."""
        docs = vectorstore.similarity_search(query, k=5)
        
        if not docs:
            return "No relevant documents found."
        
        results = []
        for i, doc in enumerate(docs, 1):
            results.append(f"[{i}] {doc.page_content[:300]}...")
        
        return "\n\n".join(results)
    
    return Tool(
        name="search_knowledge_base",
        description=description,
        func=search_documents
    )

# Usage
kb_tool = create_vector_search_tool(
    index_name="main.rag_demo.docs_vector_index",
    description="Search the internal knowledge base for product documentation, FAQs, and technical guides. Use this when answering product-related questions."
)
```

### SQL Database Query Tool

```python
from langchain.tools import Tool
from databricks import sql

def create_sql_query_tool(warehouse_id: str) -> Tool:
    """Create tool for querying Databricks SQL warehouse."""
    
    def query_database(natural_language_query: str) -> str:
        """Convert NL query to SQL and execute."""
        
        # Use LLM to generate SQL
        llm = ChatDatabricks(endpoint="databricks-meta-llama-3-1-70b-instruct")
        
        sql_prompt = f"""Generate a SQL query for this request: {natural_language_query}

Available tables:
- sales.orders (order_id, customer_id, product, amount, order_date)
- sales.customers (customer_id, name, email, region)

Return ONLY the SQL query, no explanation.
"""
        
        sql_query = llm.invoke([HumanMessage(content=sql_prompt)]).content.strip()
        
        # Remove markdown formatting if present
        sql_query = sql_query.replace('```sql', '').replace('```', '').strip()
        
        # Execute query
        with sql.connect(
            server_hostname=os.getenv("DATABRICKS_SERVER_HOSTNAME"),
            http_path=f"/sql/1.0/warehouses/{warehouse_id}",
            access_token=os.getenv("DATABRICKS_TOKEN")
        ) as connection:
            with connection.cursor() as cursor:
                cursor.execute(sql_query)
                results = cursor.fetchall()
                columns = [desc[0] for desc in cursor.description]
        
        # Format results
        if not results:
            return "No results found."
        
        # Convert to readable format
        result_str = f"Query: {sql_query}\n\nResults:\n"
        for row in results[:10]:  # Limit to 10 rows
            result_str += " | ".join([f"{col}: {val}" for col, val in zip(columns, row)]) + "\n"
        
        if len(results) > 10:
            result_str += f"\n(Showing 10 of {len(results)} results)"
        
        return result_str
    
    return Tool(
        name="query_sales_database",
        description="Query the sales database to retrieve customer orders, product information, and sales metrics. Use this for questions about order history, customer data, or sales statistics.",
        func=query_database
    )
```

### REST API Retrieval Tool

```python
import requests

def create_api_tool(api_url: str, api_key: str) -> Tool:
    """Create tool for fetching data from REST API."""
    
    def fetch_from_api(entity_id: str) -> str:
        """Retrieve entity data from external API."""
        
        headers = {"Authorization": f"Bearer {api_key}"}
        
        try:
            response = requests.get(
                f"{api_url}/entities/{entity_id}",
                headers=headers,
                timeout=10
            )
            response.raise_for_status()
            
            data = response.json()
            
            # Format response
            formatted = f"Entity ID: {data.get('id')}\n"
            formatted += f"Name: {data.get('name')}\n"
            formatted += f"Status: {data.get('status')}\n"
            formatted += f"Details: {data.get('details')}\n"
            
            return formatted
            
        except requests.exceptions.RequestException as e:
            return f"Error fetching data: {str(e)}"
    
    return Tool(
        name="fetch_entity_info",
        description="Retrieve detailed information about a specific entity from the external system using its ID.",
        func=fetch_from_api
    )
```

### Multi-Source Hybrid Retrieval Tool

```python
from typing import List, Dict

class HybridRetriever:
    def __init__(self, vectorstore, sql_connection):
        self.vectorstore = vectorstore
        self.sql_connection = sql_connection
    
    def retrieve(self, query: str, sources: List[str] = None) -> str:
        """Retrieve from multiple sources and combine results."""
        
        if sources is None:
            sources = ["vector", "sql"]
        
        results = {}
        
        # Vector search
        if "vector" in sources:
            docs = self.vectorstore.similarity_search(query, k=3)
            results["documentation"] = [doc.page_content for doc in docs]
        
        # SQL query for structured data
        if "sql" in sources:
            # Identify if query needs structured data
            if any(keyword in query.lower() for keyword in ["order", "customer", "sales", "count", "total"]):
                sql_results = self._query_sql(query)
                results["database"] = sql_results
        
        # Combine results
        combined = "Retrieved information:\n\n"
        
        if "documentation" in results:
            combined += "From Documentation:\n"
            for i, doc in enumerate(results["documentation"], 1):
                combined += f"{i}. {doc[:200]}...\n"
            combined += "\n"
        
        if "database" in results:
            combined += "From Database:\n"
            combined += results["database"] + "\n"
        
        return combined
    
    def _query_sql(self, query: str) -> str:
        # SQL query logic (simplified)
        return "Order #12345: Customer John Doe, Amount: $150"

def create_hybrid_tool(vectorstore, sql_connection) -> Tool:
    """Create hybrid retrieval tool."""
    
    retriever = HybridRetriever(vectorstore, sql_connection)
    
    return Tool(
        name="search_all_sources",
        description="Search across both documentation and database to find relevant information. Use this for comprehensive queries that may need both unstructured docs and structured data.",
        func=retriever.retrieve
    )
```

### Document-Specific Retrieval Tool

```python
def create_filtered_retrieval_tool(vectorstore, filter_key: str, filter_value: str) -> Tool:
    """Create tool that retrieves from specific document subset."""
    
    def search_filtered(query: str) -> str:
        """Search with metadata filter."""
        
        docs = vectorstore.similarity_search(
            query,
            k=5,
            filter={filter_key: filter_value}
        )
        
        if not docs:
            return f"No results found in {filter_value}."
        
        results = []
        for doc in docs:
            source = doc.metadata.get('source', 'Unknown')
            results.append(f"Source: {source}\nContent: {doc.page_content[:250]}...\n")
        
        return "\n".join(results)
    
    return Tool(
        name=f"search_{filter_value.replace(' ', '_').lower()}",
        description=f"Search specifically within {filter_value} documentation. Use this when the question is specifically about {filter_value}.",
        func=search_filtered
    )

# Create category-specific tools
databricks_tool = create_filtered_retrieval_tool(
    vectorstore, "category", "Databricks"
)
aws_tool = create_filtered_retrieval_tool(
    vectorstore, "category", "AWS"
)
```

## Best Practices

- **Clear descriptions**: LLM uses description to select tools; be specific about when to use each.
- **Error handling**: Return informative messages when retrieval fails.
- **Result formatting**: Structure output for easy LLM consumption.
- **Source attribution**: Include metadata (source file, date) in results.
- **Timeout limits**: Set timeouts for external API calls.
- **Caching**: Cache frequently accessed data to reduce latency.

## Sample Questions

1. What makes a good tool description?
2. How handle tool errors gracefully?
3. When use hybrid retrieval?
4. How limit tool scope?
5. Best practice for API tools?

## Answers

1. Specific about data source, when to use, and what queries it handles; helps LLM select correct tool.
2. Try-except blocks; return error message as string (not exception); provide fallback or guidance.
3. When queries need both unstructured (docs) and structured (DB) data; improves answer completeness.
4. Use metadata filters, separate tools per category, or restrict to specific tables/indexes.
5. Set timeouts, handle rate limits, cache responses, include authentication, validate inputs before API call.

## References

- [LangChain Tools](https://python.langchain.com/docs/modules/agents/tools/)
- [Databricks SQL Connector](https://docs.databricks.com/en/dev-tools/python-sql-connector.html)

---

Previous: [Qualitative Response Assessment](./02-qualitative-response-assessment.md)  
Next: [Prompt Formatting Effects](./04-prompt-formatting-effects.md)
