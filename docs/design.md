# Design Document: Databricks AI Agents for Home Insurance Accommodation Matching

## 1. Process Analysis

### The Problem Statement
When a homeowner's property is damaged (e.g., fire, flood, storm) and becomes uninhabitable, their home insurance policy typically includes "Loss of Use" (Coverage D) or "Additional Living Expenses" (ALE) coverage. This entitles the insured to temporary accommodation while their home is being repaired. The insurance company must find accommodation that satisfies three constraints simultaneously:
1. **Policy limits**: Daily rate caps, total ALE budget, and maximum duration.
2. **Like Kind and Quality (LKQ)**: The accommodation must be comparable to the insured's original home.
3. **Insured's specific needs**: Family size, pets, school districts, accessibility, commute.

### Key Personas

| Persona | Role | Responsibilities in the Process |
| :--- | :--- | :--- |
| **Insured (Policyholder)** | The homeowner whose property is damaged. | Files the claim, provides housing needs and preferences, approves or rejects proposed accommodation options. |
| **Claims Adjuster** | Insurance company employee who manages the claim. | Validates the claim, reviews the policy document to extract coverage limits and LKQ constraints, authorizes the accommodation budget, and oversees compliance. |
| **Relocation Vendor** | Third-party housing specialist contracted by the insurer. | Interviews the insured for specific needs, searches property inventories across multiple platforms, filters and presents accommodation options. |
| **Accommodation Provider** | Hotels, short-term rental platforms (Airbnb), or corporate housing companies. | Lists available properties with pricing, availability, and amenity details. |
| **Underwriter** | Insurance company employee responsible for policy terms. | Defines the original policy terms including Coverage D limits and LKQ definitions. (Upstream role, not active in the matching process but defines the constraints.) |

### The Current Accommodation Process

```mermaid
flowchart TD
    classDef user fill:#4A90D9,stroke:#2C6FAC,color:#FFFFFF
    classDef external fill:#9B59B6,stroke:#8E44AD,color:#FFFFFF
    classDef decision fill:#E74C3C,stroke:#C0392B,color:#FFFFFF

    A["1. Damage Event: Property becomes uninhabitable"]:::external --> B["2. Insured files Loss of Use claim"]:::user
    B --> C["3. Adjuster validates claim & reviews policy PDF"]:::user
    C --> D["4. Adjuster extracts ALE limits, daily caps & LKQ constraints"]:::user
    D --> E["5. Adjuster assigns Relocation Vendor"]:::user
    E --> F["6. Vendor interviews insured for specific needs"]:::user
    F --> G["7. Vendor searches across multiple platforms"]:::user
    G --> H["8. Vendor filters & shortlists properties"]:::user
    H --> I["9. Vendor presents options to insured"]:::user
    I --> J{Insured Approves?}:::decision
    J -- No --> G
    J -- Yes --> K["10. Adjuster reviews & authorizes booking"]:::user
    K --> L["11. Vendor books accommodation"]:::user
    L --> M["12. Ongoing: Monitor duration, extensions, budget compliance"]:::user
```

**Process Steps Explained:**
1. **Damage Event**: A covered peril (fire, flood, storm, etc.) renders the home uninhabitable.
2. **Claim Filing**: The insured contacts their insurer to file a "Loss of Use" or ALE claim.
3. **Claim Validation**: The claims adjuster verifies the claim is valid (covered peril, active policy).
4. **Policy Extraction**: The adjuster manually reads the policy PDF to determine Coverage D limits — total ALE budget, daily rate cap, maximum duration, and LKQ requirements.
5. **Vendor Assignment**: The adjuster engages a relocation vendor to handle the accommodation search.
6. **Needs Interview**: The vendor contacts the insured to gather specific housing needs (bedrooms, pets, school district, accessibility, commute).
7. **Property Search**: The vendor manually searches across fragmented platforms — hotels, Airbnb, corporate housing databases, local rental listings.
8. **Filtering & Shortlisting**: The vendor cross-references search results against both policy constraints and insured's needs to create a shortlist.
9. **Presentation**: The vendor presents the shortlisted options to the insured for review.
10. **Authorization**: Once the insured selects an option, the adjuster reviews it for policy compliance and authorizes the booking.
11. **Booking**: The vendor finalizes the reservation with the accommodation provider.
12. **Ongoing Management**: Throughout the stay, the adjuster monitors budget consumption, handles extension requests if repairs take longer, and ensures ongoing compliance with policy terms.

### Pain Points Identified
1. **Manual Policy Parsing**: Adjusters spend significant time reading dense, unstructured PDFs to find specific sub-limits and LKQ definitions.
2. **Slow Turnaround**: The end-to-end process from claim to booking can take days, leaving displaced families without housing.
3. **Fragmented Search**: Vendors search across disconnected platforms one by one, often missing the best options.
4. **Suboptimal Matching**: Human error in cross-referencing policy limits with property details leads to bookings that exceed budgets or fail LKQ requirements.
5. **Poor Ongoing Tracking**: Budget consumption and duration compliance are tracked manually, creating risk of overruns.
6. **High Vendor Costs**: Third-party relocation vendors charge significant fees for what is largely a manual search-and-coordinate workflow.

---

## 2. Proposed AI Agents Solution on Databricks

To automate and optimize this workflow, we propose a Multi-Agent System built natively on the Databricks Data Intelligence Platform. Databricks provides the necessary tools for secure data governance (Unity Catalog), vector storage, and agent serving (Mosaic AI Agent Framework).

### Pain Point to Agent Capability Mapping

Each pain point from Section 1 maps to a specific AI agent capability:

| Pain Point | AI Capability | Responsible Agent |
| :--- | :--- | :--- |
| Manual Policy Parsing | RAG — retrieve and extract from unstructured PDFs | Policy Analyzer Agent |
| Slow Turnaround | Parallel execution — all agents work simultaneously | Matching Orchestrator |
| Fragmented Search | Tool calling — query multiple APIs in parallel | Property Broker Agent |
| Suboptimal Matching | Scoring algorithm — deterministic rank by cost and fit | Matching Orchestrator |
| Poor Ongoing Tracking | Automated monitoring — scheduled budget/duration checks | Compliance Monitor (future) |
| High Vendor Costs | End-to-end automation — reduces vendor dependency | All Agents |

### Proposed Agentic Workflow

```mermaid
sequenceDiagram
    box rgb(74, 144, 217) Human Actors
        participant Insured as Insured Person
        participant Adjuster as Claims Adjuster
    end
    box rgb(46, 204, 113) AI Agents
        participant Orchestrator as Matching Orchestrator
        participant Analyzer as Policy Analyzer Agent
        participant Intake as Needs Intake Agent
        participant Broker as Property Broker Agent
    end
    box rgb(243, 156, 18) Data Layer
        participant UC as Unity Catalog
    end
    box rgb(155, 89, 182) External APIs
        participant ExtAPI as External APIs
    end

    Insured->>Adjuster: Files Loss of Use claim
    Adjuster->>Orchestrator: Initiates matching (Policy ID, Claim ID)

    par Policy Analysis
        Orchestrator->>Analyzer: Extract policy constraints
        Analyzer->>UC: Vector Search on policy PDF chunks
        UC-->>Analyzer: Relevant policy clauses
        Analyzer-->>Orchestrator: Structured constraints (ALE limit, daily cap, LKQ)
    and Needs Assessment
        Orchestrator->>Intake: Gather insured needs
        Intake->>Insured: "Tell us about your housing needs"
        Insured-->>Intake: "3 beds, dog friendly, near Lincoln High"
        Intake-->>Orchestrator: Structured needs JSON
    end

    Orchestrator->>Broker: Search properties (constraints + needs)
    par Internal Search
        Broker->>UC: Query inventory Delta tables
        UC-->>Broker: Internal property results
    and External Search
        Broker->>ExtAPI: Query Airbnb, Hotels, Corporate Housing
        ExtAPI-->>Broker: External property results
    end
    Broker-->>Orchestrator: Combined raw property list

    Orchestrator->>Orchestrator: Filter hard constraints, score and rank
    Orchestrator->>Adjuster: Present top 3 matches for authorization
    Adjuster->>Insured: Present approved options
    Insured-->>Adjuster: Selects preferred option
    Adjuster->>Orchestrator: Confirm booking
```

**Key design decisions in this workflow:**
1. **Human-in-the-loop**: The Claims Adjuster remains in the loop for authorization. The agents assist, but do not autonomously book accommodation.
2. **Parallel execution**: Policy analysis and needs intake happen concurrently (`par` block) to reduce turnaround time.
3. **Internal + External search**: The Broker queries both Unity Catalog inventory tables and external APIs in parallel.

---

## 3. Solution Architecture Design

### System Architecture

```mermaid
graph TD
    classDef user fill:#4A90D9,stroke:#2C6FAC,color:#FFFFFF
    classDef agent fill:#2ECC71,stroke:#27AE60,color:#FFFFFF
    classDef datastore fill:#F39C12,stroke:#E67E22,color:#FFFFFF
    classDef external fill:#9B59B6,stroke:#8E44AD,color:#FFFFFF
    classDef monitoring fill:#95A5A6,stroke:#7F8C8D,color:#FFFFFF

    subgraph Legacy Systems
        ClaimsDB[("Legacy Claims DB")]:::datastore
        PolicyPDFs[("Policy PDF Storage")]:::datastore
    end

    subgraph Databricks - Data Layer
        DLT["Delta Live Tables (Ingestion)"]:::datastore
        UC_Claims[("claims.policies")]:::datastore
        UC_Inventory[("accommodations.inventory")]:::datastore
        UC_Volumes[("claims.policy_documents (Volumes)")]:::datastore
        VectorIdx[("Vector Search Index")]:::datastore
    end

    subgraph Databricks - Agent Layer
        Orchestrator["Matching Orchestrator (LangGraph)"]:::agent
        Analyzer["Policy Analyzer (RAG)"]:::agent
        Intake["Needs Intake (Conversational)"]:::agent
        Broker["Property Broker (Tool Calling)"]:::agent
        ModelServing["Mosaic AI Model Serving"]:::agent
    end

    subgraph External
        Airbnb["Airbnb API"]:::external
        Hotels["Hotel APIs"]:::external
        CorpHousing["Corporate Housing API"]:::external
        UserUI["Insured / Adjuster UI"]:::user
    end

    subgraph Observability
        MLflow["MLflow Tracing & Evaluation"]:::monitoring
    end

    ClaimsDB -->|batch sync| DLT
    PolicyPDFs -->|ingestion| UC_Volumes
    DLT --> UC_Claims
    DLT --> UC_Inventory
    UC_Volumes -->|chunk & embed| VectorIdx

    UserUI <--> Orchestrator
    Orchestrator <--> Analyzer
    Orchestrator <--> Intake
    Orchestrator <--> Broker

    Analyzer <--> VectorIdx
    Analyzer <--> ModelServing
    Intake <--> ModelServing
    Broker <--> UC_Inventory
    Broker <--> Airbnb
    Broker <--> Hotels
    Broker <--> CorpHousing

    Orchestrator --> MLflow
    Analyzer --> MLflow
    Broker --> MLflow
```

### Agent Specifications

#### 1. Policy Analyzer Agent (RAG)

| Attribute | Detail |
| :--- | :--- |
| **Role** | Retrieves relevant sections from the insured's policy PDF and extracts structured financial constraints. |
| **Pattern** | RAG (Retrieval-Augmented Generation) |
| **Databricks Components** | Unity Catalog Volumes (PDF storage), Databricks Vector Search (retrieval), Mosaic AI Model Serving (LLM) |
| **Input** | `policy_id` (string) |
| **Output** | `PolicyConstraints` — total ALE limit, daily rate cap, max duration, LKQ flag, LKQ description |
| **Core Logic** | 1. Retrieve top-k chunks from Vector Search matching "Additional Living Expenses" and "Loss of Use". 2. Prompt the LLM with retrieved context to extract structured fields. 3. Return validated `PolicyConstraints` object. |

#### 2. Needs Intake Agent (Conversational)

| Attribute | Detail |
| :--- | :--- |
| **Role** | Interviews the insured via chat to capture their specific accommodation requirements. |
| **Pattern** | Conversational with structured output |
| **Databricks Components** | Mosaic AI Model Serving (LLM with function calling) |
| **Input** | Chat history (list of messages) |
| **Output** | `UserNeeds` — bedrooms, pets, school district, accessibility, commute, special requests |
| **Core Logic** | 1. Use a system prompt to guide empathetic, structured conversation. 2. Track which required fields are still missing. 3. Once all required fields are captured, output structured JSON via function calling. |

#### 3. Property Broker Agent (Tool Calling)

| Attribute | Detail |
| :--- | :--- |
| **Role** | Searches for available accommodation from internal inventory and external providers. |
| **Pattern** | Tool-calling agent |
| **Databricks Components** | Mosaic AI Agent Framework, Unity Catalog Functions (tools), Databricks SQL |
| **Input** | `PolicyConstraints` + `UserNeeds` |
| **Output** | List of `PropertyMatch` — property ID, address, bedrooms, pet policy, daily rate, provider |
| **Tools** | `search_internal_inventory(location, beds, pet_friendly, max_daily_rate)` — UC Function querying `accommodations.inventory`. `search_external_api(provider, location, beds, pet_friendly, max_daily_rate)` — UC Function wrapping external API calls. |
| **Core Logic** | 1. Convert needs and constraints into tool call parameters. 2. Execute internal and external searches in parallel. 3. Deduplicate and return combined results. |

#### 4. Matching Orchestrator (Coordinator)

| Attribute | Detail |
| :--- | :--- |
| **Role** | Coordinates the full workflow, applies hard filters, scores properties, and presents ranked results. |
| **Pattern** | LangGraph orchestrator |
| **Databricks Components** | Mosaic AI Agent Framework, Databricks Workflows (trigger) |
| **Input** | `policy_id`, `claim_id`, chat session |
| **Output** | Top 3 ranked `PropertyMatch` results |
| **Core Logic** | 1. Dispatch Policy Analyzer and Needs Intake in parallel. 2. Pass combined constraints + needs to Property Broker. 3. **Hard filter**: Eliminate properties that violate constraints (over daily cap, insufficient bedrooms, not pet friendly when pets required). 4. **Score**: `suitability_score = (0.5 × need_match) + (0.3 × cost_efficiency) + (0.2 × lkq_match)`. 5. Return top 3 by score. |

### Data Model (Unity Catalog)

**Catalog**: `insurance_ai`

#### Schema: `claims`

**Table: `policies`**
| Column | Type | Description |
| :--- | :--- | :--- |
| `policy_id` | STRING | Primary key |
| `customer_id` | STRING | FK to customer |
| `dwelling_coverage_usd` | DOUBLE | Coverage A amount |
| `policy_document_path` | STRING | Path to PDF in UC Volumes |
| `effective_date` | DATE | Policy start date |
| `expiry_date` | DATE | Policy end date |

**Table: `extracted_policy_constraints`**
| Column | Type | Description |
| :--- | :--- | :--- |
| `policy_id` | STRING | FK to policies |
| `total_ale_limit_usd` | DOUBLE | Max ALE budget |
| `daily_limit_usd` | DOUBLE | Max daily accommodation rate |
| `lkq_required` | BOOLEAN | LKQ constraint flag |
| `lkq_description` | STRING | Free-text LKQ definition from policy |
| `max_duration_months` | INT | Max temporary housing duration |
| `extracted_at` | TIMESTAMP | When extraction was performed |

#### Schema: `accommodations`

**Table: `inventory`**
| Column | Type | Description |
| :--- | :--- | :--- |
| `property_id` | STRING | Primary key |
| `provider_name` | STRING | e.g., Airbnb, Corporate Housing |
| `address` | STRING | Full property address |
| `city` | STRING | City |
| `state` | STRING | State |
| `bedrooms` | INT | Number of bedrooms |
| `bathrooms` | INT | Number of bathrooms |
| `pet_friendly` | BOOLEAN | Accepts pets |
| `daily_rate_usd` | DOUBLE | Current daily rate |
| `available_from` | DATE | Start of availability |
| `available_to` | DATE | End of availability |
| `last_updated` | TIMESTAMP | Last inventory refresh |

**Table: `match_results`**
| Column | Type | Description |
| :--- | :--- | :--- |
| `match_id` | STRING | Primary key |
| `claim_id` | STRING | FK to claim |
| `policy_id` | STRING | FK to policy |
| `property_id` | STRING | FK to inventory |
| `suitability_score` | DOUBLE | Calculated score |
| `status` | STRING | PROPOSED / APPROVED / BOOKED / REJECTED |
| `created_at` | TIMESTAMP | When match was generated |

### Legacy Integration Strategy

| Legacy System | Integration Method | Databricks Target |
| :--- | :--- | :--- |
| Claims Database | Delta Live Tables (batch or streaming CDC) | `claims.policies` Delta table |
| Policy PDF Storage | File ingestion pipeline | `claims.policy_documents` UC Volume |
| Accommodation Provider Feeds | Scheduled API ingestion via Databricks Workflows | `accommodations.inventory` Delta table |

### Agent Evaluation Plan

Before deploying any agent to production, use **Mosaic AI Agent Evaluation** to validate quality:

| Agent | Evaluation Approach |
| :--- | :--- |
| Policy Analyzer | Compare extracted constraints against manually labeled ground truth for 50+ policies. Measure field-level accuracy. |
| Needs Intake | Validate that structured output correctly captures all stated needs from sample conversations. |
| Property Broker | Verify that returned properties match the query parameters (no constraint violations in results). |
| Matching Orchestrator | End-to-end test: given a policy + needs, verify top 3 results are valid, correctly scored, and ordered. |

All evaluation metrics are tracked in **MLflow** for reproducibility.
