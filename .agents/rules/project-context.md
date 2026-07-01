# Project Context: Home Insurance Accommodation Matching

## Domain
This project is in the **home insurance** domain, specifically focused on the "Loss of Use" (Coverage D) / "Additional Living Expenses" (ALE) workflow. The core problem is matching displaced policyholders with temporary accommodation that satisfies policy constraints and personal needs.

## Key Domain Terminology
When working on this project, use these terms precisely:
- **ALE (Additional Living Expenses)**: The coverage that pays for temporary housing when the insured's home is uninhabitable.
- **Coverage D (Loss of Use)**: The section of a homeowner's policy that includes ALE.
- **LKQ (Like Kind and Quality)**: The policy requirement that temporary accommodation must be comparable to the insured's original home.
- **Insured / Policyholder**: The homeowner who holds the insurance policy.
- **Claims Adjuster**: The insurance company employee who validates and manages the claim.
- **Relocation Vendor**: A third-party specialist who searches for and coordinates accommodation.

## Project Structure
- `docs/` — All design documents. Every new feature or component must have a design document here before code is written.
- `src/` — Source code. Only created after designs are approved.
- `.agents/` — AI agent configuration (rules, skills).

## Design Document Standards
When writing design documents in `docs/`:
1. Always analyze the current process before proposing a solution.
2. Identify and list all personas involved in the process.
3. Use Mermaid diagrams for process flows and architecture diagrams.
4. Identify pain points in the current process and map them to proposed agent capabilities.
5. Design documents must be concise, comprehensive, and written in Markdown.

## Platform
The target platform is **Databricks**. All solutions should use Databricks-native capabilities:
- **Unity Catalog** for data governance and storage.
- **Delta Live Tables (DLT)** for data ingestion pipelines.
- **Mosaic AI Agent Framework** for authoring and deploying agents.
- **Databricks Vector Search** for RAG-based document retrieval.
- **Mosaic AI Model Serving** for LLM inference.
- **MLflow** for experiment tracking and agent tracing.
- **Databricks Workflows** for orchestration and scheduling.
