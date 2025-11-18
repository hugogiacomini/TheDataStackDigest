# Batch Inference Workloads with ai_query()

## Overview

Execute batch LLM inference on large datasets using Databricks SQL's `ai_query()` function: process thousands of rows efficiently without deploying endpoints.

## ai_query() Function

**Syntax**:

```sql
ai_query(
  endpoint_name STRING,
  request STRUCT|STRING,
  [returnType STRING]
) RETURNS returnType
```

**Use Cases**:

- Batch text classification
- Bulk summarization
- Data enrichment at scale
- Embedding generation for datasets

## Hands-on Examples

### Basic Batch Classification

```sql
-- Classify support tickets by sentiment
CREATE OR REPLACE TABLE support_tickets_classified AS
SELECT 
    ticket_id,
    customer_message,
    ai_query(
        'databricks-meta-llama-3-1-70b-instruct',
        CONCAT(
            'Classify sentiment as POSITIVE, NEGATIVE, or NEUTRAL: ',
            customer_message
        )
    ) as sentiment
FROM support_tickets
WHERE timestamp >= current_date - INTERVAL 7 DAYS;

SELECT * FROM support_tickets_classified LIMIT 10;
```

### Structured Output with JSON

```sql
-- Extract entities from text with structured output
SELECT 
    ticket_id,
    ai_query(
        'databricks-meta-llama-3-1-70b-instruct',
        CONCAT(
            'Extract person name, email, and issue category as JSON: ',
            customer_message,
            '\nFormat: {"name": "...", "email": "...", "category": "..."}'
        )
    ) as extracted_entities
FROM support_tickets
LIMIT 100;
```

### PySpark with ai_query()

```python
from pyspark.sql.functions import expr

# Load data
df = spark.table("main.support.customer_tickets")

# Apply ai_query for batch processing
classified_df = df.withColumn(
    "priority",
    expr("""
        ai_query(
            'databricks-meta-llama-3-1-70b-instruct',
            concat(
                'Classify ticket priority as HIGH, MEDIUM, or LOW. ',
                'Consider urgency and impact. Ticket: ',
                description
            )
        )
    """)
)

# Write results
classified_df.write.mode("overwrite").saveAsTable("main.support.tickets_prioritized")
```

### Batch Summarization

```python
from pyspark.sql.functions import col

# Summarize long documents
articles_df = spark.table("main.content.articles")

summaries_df = articles_df.select(
    col("article_id"),
    col("title"),
    expr("""
        ai_query(
            'databricks-meta-llama-3-1-70b-instruct',
            concat(
                'Summarize in 2-3 sentences: ',
                substring(content, 1, 4000)
            )
        )
    """).alias("summary")
)

summaries_df.write.mode("overwrite").saveAsTable("main.content.article_summaries")
```

### Generate Embeddings at Scale

```python
# Generate embeddings for all documents
from pyspark.sql.functions import pandas_udf, col
from pyspark.sql.types import ArrayType, DoubleType
import pandas as pd

@pandas_udf(ArrayType(DoubleType()))
def generate_embeddings_batch(texts: pd.Series) -> pd.Series:
    """Generate embeddings using Databricks endpoint."""
    from databricks_genai import Embedding
    
    embeddings_model = Embedding(endpoint="databricks-bge-large-en")
    
    # Batch process
    results = []
    for text in texts:
        embedding = embeddings_model.embed(text[:512])  # Truncate to model max
        results.append(embedding.tolist())
    
    return pd.Series(results)

# Apply to dataset
docs_df = spark.table("main.rag_demo.document_chunks")

embedded_df = docs_df.withColumn(
    "embedding",
    generate_embeddings_batch(col("content"))
)

# Write to Delta with embeddings
embedded_df.write.format("delta").mode("overwrite").saveAsTable(
    "main.rag_demo.document_chunks_embedded"
)
```

### Batch Translation

```sql
-- Translate product descriptions to multiple languages
CREATE OR REPLACE TABLE product_descriptions_translated AS
SELECT 
    product_id,
    description_en,
    ai_query(
        'databricks-meta-llama-3-1-70b-instruct',
        CONCAT('Translate to Spanish: ', description_en)
    ) as description_es,
    ai_query(
        'databricks-meta-llama-3-1-70b-instruct',
        CONCAT('Translate to French: ', description_en)
    ) as description_fr
FROM products;
```

### Cost-Optimized Batch Processing

```python
from pyspark.sql.functions import when, length

# Only process rows that need inference
df = spark.table("main.support.tickets")

# Filter: only classify if not already classified and text is substantial
to_classify = df.filter(
    (col("sentiment").isNull()) & 
    (length(col("message")) > 50)
)

# Process in batches with repartitioning for efficiency
classified = to_classify.repartition(100).select(
    col("ticket_id"),
    col("message"),
    expr("""
        ai_query(
            'databricks-meta-llama-3-1-8b-instruct',
            concat('Sentiment (POSITIVE/NEGATIVE/NEUTRAL): ', message)
        )
    """).alias("sentiment")
)

# Merge back
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(spark, "main.support.tickets")

delta_table.alias("target").merge(
    classified.alias("source"),
    "target.ticket_id = source.ticket_id"
).whenMatchedUpdate(
    set={"sentiment": "source.sentiment"}
).execute()
```

### Monitoring Batch Jobs

```python
# Track progress and costs
import time

start_time = time.time()

# Execute batch job
result_df = df.withColumn(
    "classification",
    expr("ai_query('databricks-meta-llama-3-1-70b-instruct', prompt_column)")
)

result_count = result_df.count()
duration = time.time() - start_time

# Log metrics
print(f"Processed {result_count} rows in {duration:.2f} seconds")
print(f"Throughput: {result_count/duration:.2f} rows/sec")

# Estimate cost (approximate)
avg_tokens_per_row = 200  # Estimate based on prompt + response
total_tokens = result_count * avg_tokens_per_row
cost_per_1k_tokens = 0.001  # Llama 70B pricing
estimated_cost = (total_tokens / 1000) * cost_per_1k_tokens

print(f"Estimated cost: ${estimated_cost:.2f}")
```

### Error Handling in Batch

```python
from pyspark.sql.functions import expr, col

# Wrap ai_query with error handling
result_df = df.withColumn(
    "result",
    expr("""
        TRY(
            ai_query(
                'databricks-meta-llama-3-1-70b-instruct',
                prompt_text
            )
        )
    """)
).withColumn(
    "has_error",
    col("result").isNull()
)

# Separate successful and failed rows
successful = result_df.filter(col("has_error") == False)
failed = result_df.filter(col("has_error") == True)

print(f"Successful: {successful.count()}, Failed: {failed.count()}")

# Retry failed rows with smaller model
retried = failed.withColumn(
    "result_retry",
    expr("""
        ai_query(
            'databricks-meta-llama-3-1-8b-instruct',
            prompt_text
        )
    """)
)
```

## Performance Optimization

### Partition for Parallelism

```python
# Repartition for better parallelism
optimized_df = df.repartition(200).withColumn(
    "inference_result",
    expr("ai_query('endpoint', prompt)")
)
```

### Cache Repeated Prompts

```python
# Deduplicate before inference
unique_prompts = df.select("prompt").distinct()

inferred = unique_prompts.withColumn(
    "result",
    expr("ai_query('endpoint', prompt)")
)

# Join back to original
final_df = df.join(inferred, "prompt")
```

## Best Practices

- **Use for batch only**: Not for real-time (use endpoints for low-latency).
- **Appropriate model size**: Use smaller models (8B) for simple tasks to reduce cost.
- **Partition data**: Repartition for parallelism; typical: 100-200 partitions.
- **Handle errors**: Wrap in TRY() to handle individual row failures.
- **Monitor costs**: Track token usage; batch can be expensive at scale.
- **Incremental processing**: Only process new/unprocessed rows.

## Sample Questions

1. When use ai_query() vs endpoint?
2. How handle errors in batch inference?
3. How optimize cost for batch jobs?
4. Typical throughput for ai_query()?
5. Can ai_query() generate embeddings?

## Answers

1. ai_query() for batch (thousands of rows, not time-sensitive); endpoint for real-time (low latency, user-facing).
2. Wrap in TRY() to return NULL on error; filter and retry failed rows; log failures for investigation.
3. Use smaller models for simple tasks, deduplicate prompts, incremental processing, cache results, use shorter prompts.
4. Varies by model/cluster; typically 10-100 rows/sec for 70B model; 8B can do 100-500 rows/sec.
5. Yes, but prefer dedicated embedding models (databricks-bge-large-en) via ai_query() or pandas_udf with SDK.

## References

- [Databricks ai_query()](https://docs.databricks.com/en/large-language-models/ai-functions.html)
- [Batch Inference Best Practices](https://docs.databricks.com/en/machine-learning/model-serving/score-model-serving-endpoints.html)

---

Previous: [Endpoint Access Control](./04-endpoint-access-control.md)  
Next: [Governance: Masking Techniques](../05-governance/01-masking-techniques-guardrails.md)
