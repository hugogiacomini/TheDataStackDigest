# Databricks Generative AI Engineer Associate – Study Guide

## Overview

Comprehensive exam-aligned study guide for the Databricks Generative AI Engineer Associate certification. Covers RAG application design, data preparation, development, deployment, governance, and evaluation.

## Exam Scope

- **Design Applications**: Prompt engineering, model selection, chain composition, multi-stage reasoning
- **Data Preparation**: Chunking, filtering, extraction, Delta Lake integration, retrieval evaluation
- **Application Development**: Tools creation, LangChain, guardrails, model selection, agents
- **Assembling & Deploying**: Chains, pyfunc models, Vector Search, Model Serving, MLflow
- **Governance**: Guardrails, masking, legal compliance, data quality
- **Evaluation & Monitoring**: Metrics, MLflow evaluation, inference logging, cost control

## Audience

Data engineers and ML engineers building production GenAI/RAG applications on Databricks with foundational LLM knowledge.

## Prerequisites

- Databricks platform basics (notebooks, jobs, clusters)
- Python proficiency (pandas, PySpark basics)
- Understanding of ML concepts (embeddings, similarity search)
- Familiarity with Unity Catalog and Delta Lake

## Section Index

### 01. Design Applications

- 01 [Prompt Engineering for Formatted Responses](./01-design-applications/01-prompt-engineering-formatted-responses.md)
- 02 [Model Task Selection for Business Requirements](./01-design-applications/02-model-task-selection.md)
- 03 [Chain Component Selection](./01-design-applications/03-chain-component-selection.md)
- 04 [Translating Business Goals to AI Pipeline Specs](./01-design-applications/04-business-goals-to-pipeline-specs.md)
- 05 [Multi-Stage Reasoning with Tools and Agents](./01-design-applications/05-multi-stage-reasoning-tools.md)

### 02. Data Preparation

- 01 [Chunking Strategies for Document Structure](./02-data-preparation/01-chunking-strategies.md)
- 02 [Filtering Extraneous Content](./02-data-preparation/02-filtering-extraneous-content.md)
- 03 [Document Content Extraction by Format](./02-data-preparation/03-document-content-extraction.md)
- 04 [Writing Chunks to Delta Lake in Unity Catalog](./02-data-preparation/04-writing-chunks-to-delta-lake.md)
- 05 [Source Document Selection for RAG Quality](./02-data-preparation/05-source-document-selection.md)
- 06 [Prompt/Response Pairs for Model Tasks](./02-data-preparation/06-prompt-response-pairs.md)
- 07 [Retrieval Performance Evaluation](./02-data-preparation/07-retrieval-performance-evaluation.md)
- 08 [Advanced Chunking Strategies](./02-data-preparation/08-advanced-chunking-strategies.md)
- 09 [Re-Ranking in Information Retrieval](./02-data-preparation/09-reranking-in-retrieval.md)
- 10 [Chunking Strategy Selection Based on Evaluation](./02-data-preparation/10-chunking-strategy-selection.md)

### 03. Application Development

- 01 [Implementing LLM Guardrails to Prevent Negative Outcomes](./03-application-development/01-implementing-llm-guardrails.md)
- 02 [Qualitative Response Assessment for LLM Outputs](./03-application-development/02-qualitative-response-assessment.md)
- 03 [Creating Data Retrieval Tools for Specific Needs](./03-application-development/03-creating-data-retrieval-tools.md)
- 04 [Identifying Prompt Formatting Effects on Model Outputs](./03-application-development/04-prompt-formatting-effects.md)
- 05 [Writing Metaprompts to Minimize Hallucinations and Data Leakage](./03-application-development/05-writing-metaprompts-anti-hallucination.md)
- 06 [Model Selection Based on Application Requirements](./03-application-development/06-model-selection-requirements.md)
- 07 [Utilizing Databricks Agent Framework for Agentic Systems](./03-application-development/07-agent-framework-utilization.md)

### 04. Assembling and Deploying Applications

- 01 [Coding RAG Chains with PyFunc Models (Pre/Post-Processing)](./04-assembling-deploying-applications/01-pyfunc-models-rag.md)
- 02 [Vector Search Index Creation and Querying](./04-assembling-deploying-applications/02-vector-search-index-creation.md)
- 03 [Sequencing Endpoint Deployment Steps for RAG Applications](./04-assembling-deploying-applications/03-model-serving-endpoint-deployment.md)
- 04 [Controlling Access to Model Serving Endpoint Resources](./04-assembling-deploying-applications/04-endpoint-access-control.md)
- 05 [Batch Inference Workloads with ai_query()](./04-assembling-deploying-applications/05-batch-inference-ai-query.md)

### 05. Governance

- 01 [Masking Techniques as Guardrails for Performance Objectives](./05-governance/01-masking-techniques-guardrails.md)
- 02 [Protecting Against Malicious User Inputs with Guardrails](./05-governance/02-protecting-malicious-inputs.md)
- 03 [Using Legal and Licensing Requirements to Avoid Legal Risk](./05-governance/03-legal-licensing-compliance.md)

### 06. Evaluation and Monitoring

- 01 [RAG Performance Evaluation with MLflow](./06-evaluation-monitoring/01-rag-evaluation-mlflow.md)
- 02 [Inference Logging and Performance Monitoring](./06-evaluation-monitoring/02-inference-logging-monitoring.md)
- 03 [Using Databricks Features to Control LLM Costs](./06-evaluation-monitoring/03-cost-control-features.md)

## Study Approach

1. **Sequential Learning**: Progress through sections; foundational concepts build on each other.
2. **Hands-on Practice**: Reproduce code examples in Databricks notebooks; modify parameters to observe behavior.
3. **Build Mini-Projects**:
   - Simple RAG: PDF → chunks → embeddings → Vector Search → LLM
   - Agent: Multi-tool reasoning for customer support
   - Monitored deployment: Track inference quality and costs
4. **Evaluate Thoroughly**: Use RAGAS, MLflow; understand metric interpretation.
5. **Governance Integration**: Apply guardrails and masking from the start; not as afterthought.
6. **Review Sample Questions**: Each file includes 5 Q&A pairs covering key concepts.

## Key Technologies

- **Databricks Platform**: Workspace, Model Serving, Vector Search, Unity Catalog
- **Frameworks**: LangChain, MLflow, RAGAS, Unstructured.io
- **Models**: Llama 3.1, BGE embeddings, Databricks Foundation Models
- **Storage**: Delta Lake, Unity Catalog (governance)
- **Development**: Python, PySpark, pandas

## Recommended Resources

- [Databricks GenAI Documentation](https://docs.databricks.com/en/generative-ai/index.html)
- [LangChain Documentation](https://python.langchain.com/)
- [MLflow Model Evaluation](https://mlflow.org/docs/latest/model-evaluation/index.html)
- [RAGAS Framework](https://docs.ragas.io/)
- [Databricks Academy](https://www.databricks.com/learn/training)

## Exam Tips

- Focus on **Databricks-specific implementations** (Vector Search, Model Serving, Unity Catalog integration).
- Understand **trade-offs**: chunk size vs context, retrieval speed vs precision, cost vs accuracy.
- Know **when to use** each technique (guardrails, re-ranking, agents) based on requirements.
- Be familiar with **evaluation metrics**: Precision@K, MRR, NDCG, faithfulness, answer relevancy.
- Practice **code patterns**: pyfunc models, LangChain chains, embedding generation, Delta writes.

---

**Start**: [Prompt Engineering for Formatted Responses](./01-design-applications/01-prompt-engineering-formatted-responses.md)
