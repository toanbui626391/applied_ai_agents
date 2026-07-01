---
name: design-document
description: Skill for writing design documents for AI agent projects. Triggers when the user asks to design, architect, or document a feature, process, or system.
---

# Design Document Skill

Use this skill when creating or updating design documents in the `docs/` directory.

## Document Structure

Every design document must follow this structure:

### 1. Process Analysis
- **Problem Statement**: Clearly define the problem being solved in 2-3 sentences.
- **Key Personas**: List all personas involved in the process as a table with columns: Persona, Role, Responsibilities.
- **Current Process**: Draw the current (as-is) process using a Mermaid `flowchart TD` diagram. Number each step.
- **Process Steps Explained**: Describe each numbered step in 1-2 sentences.
- **Pain Points**: List specific inefficiencies, risks, and costs in the current process.

### 2. Proposed AI Agents Solution
- **Where AI Agents Add Value**: Map each pain point to a specific AI capability (RAG, tool calling, conversational AI, optimization).
- **Proposed Agentic Workflow**: Draw the proposed (to-be) workflow using a Mermaid `sequenceDiagram` showing agent interactions.

### 3. Solution Architecture
- **System Architecture Diagram**: Use a Mermaid `graph TD` diagram showing all components and data flows.
- **Agent Specifications**: For each agent, document: Role, Tech Stack, Input, Output, and Core Logic.
- **Data Model**: Define the key tables/schemas as Markdown tables.
- **Legacy Integration Strategy**: Describe how the solution connects to existing systems.

## Mermaid Diagram Guidelines
- Use `flowchart TD` for process flows (top-down).
- Use `sequenceDiagram` for agent interaction sequences.
- Use `graph TD` for architecture diagrams.
- Quote node labels that contain special characters: `id["Label with (parens)"]`.
- Keep labels concise — no more than 8-10 words per node.

## Writing Style
- Be concise but comprehensive.
- Use domain-specific terminology correctly (ALE, LKQ, Coverage D).
- Number process steps for easy reference.
- Always validate the process analysis by asking: "Does this cover the full lifecycle from trigger event to ongoing management?"
