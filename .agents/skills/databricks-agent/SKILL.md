---
name: databricks-agent
description: Skill for designing and implementing AI agents on the Databricks platform. Triggers when building agents, RAG pipelines, or agentic workflows using Databricks Mosaic AI.
---

# Databricks AI Agent Development Skill

Use this skill when designing or implementing AI agents on the Databricks Data Intelligence Platform.

## Databricks-Native Agent Stack

Always prefer these Databricks-native components over generic alternatives:

| Capability | Databricks Component | Avoid |
| :--- | :--- | :--- |
| Data Storage & Governance | Unity Catalog (Delta Tables, Volumes) | Raw S3/ADLS, unmanaged Parquet |
| Vector Database | Databricks Vector Search | Pinecone, Weaviate, ChromaDB |
| LLM Serving | Mosaic AI Model Serving | Self-hosted vLLM, OpenAI direct |
| Agent Framework | Mosaic AI Agent Framework | Standalone LangChain deployments |
| Data Pipelines | Delta Live Tables (DLT) | Custom Spark jobs, Airflow |
| Orchestration | Databricks Workflows | Airflow, Step Functions |
| Experiment Tracking | MLflow (built-in) | Weights & Biases, Neptune |
| Agent Tracing | MLflow Tracing | LangSmith |

## Agent Design Patterns

### RAG Agent (e.g., Policy Analyzer)
1. Store source documents (PDFs) in **Unity Catalog Volumes**.
2. Chunk and embed documents using a Databricks notebook pipeline.
3. Sync embeddings to a **Databricks Vector Search** index.
4. Serve the retrieval + generation chain via **Mosaic AI Model Serving**.

### Tool-Calling Agent (e.g., Property Broker)
1. Define tools as **Unity Catalog Functions** (SQL or Python).
2. Register tools with the agent via the **Mosaic AI Agent Framework**.
3. The LLM decides which tools to call based on the user query.

### Conversational Agent (e.g., Intake & Triage)
1. Use structured output / function calling to extract information from conversation.
2. Define the output schema as a Pydantic model.
3. Maintain conversation state for multi-turn interactions.

### Orchestrator Agent (e.g., Matching Orchestrator)
1. Use a **LangGraph** or **ReAct** pattern for multi-step reasoning.
2. The orchestrator delegates to sub-agents (Analyzer, Intake, Broker).
3. Implement scoring/ranking logic as a deterministic function, not LLM-generated.

## Unity Catalog Conventions
- **Catalog**: Use a project-level catalog (e.g., `insurance_ai`).
- **Schemas**: Group by domain (e.g., `claims`, `accommodations`, `agents`).
- **Tables**: Use Delta format. Name tables as `snake_case` nouns (e.g., `extracted_policy_constraints`).
- **Volumes**: Store unstructured data (PDFs, images) in managed volumes.

## Agent Evaluation
- Use **Mosaic AI Agent Evaluation** to test agent quality before deployment.
- Define evaluation datasets with expected inputs and outputs.
- Track evaluation metrics in **MLflow**.
