# Solution Architecture

## High Level Architecture Breakdown

Main sections:

1. Presentation Level (streamlit / JS)

3. BFF Level (FastAPI / orchestration)
3. Synchronization Level (Agent Tools / CrewAI or pydantic.ai)
4. Recommendation Engine (Multi-Agentic / CrewAI or pydantic.ai)
5. Calculation Engine (Agent Tools / CrewAI or pydantic.ai)

6. Persistence (Postgres / pgvector)


## Orchestration and Muti-Agent Architecture

graph TD
    %% Presentation and Entry
    User((User / Frontend)) -->|API Request| BFF[FastAPI BFF / Orchestrator]
    
    subgraph "Orchestration Layer (FastAPI)"
        BFF --> Router{Task Router}
        Router -->|Sync Data| SyncAgent[Synchronization Agent]
        Router -->|Analyze/Recommend| RecAgent[Recommendation Engine]
        Router -->|Compute| CalcAgent[Calculation Engine]
    end

    subgraph "Agentic Framework (CrewAI / PydanticAI)"
        SyncAgent -->|Tool Use| DB_Connector[(External Sources)]
        
        RecAgent -->|Collaborates| ManagerAgent[Manager / Supervisor Agent]
        ManagerAgent --> SearchTool[RAG / Vector Search]
        ManagerAgent --> StrategyTool[Strategy Analyzer]
        
        CalcAgent -->|Tool Use| MathTool[Python Interpreter / Calculator]
    end

    subgraph "Persistence Layer"
        SyncAgent -->|Upsert| PG[(Postgres / pgvector)]
        RecAgent -->|Query| PG
        CalcAgent -->|Store Snapshots| PG
    end

    %% Feedback Loop
    PG -.->|Context/State| BFF
    RecAgent -.->|Response| BFF
    BFF -->|JSON/Stream| User


## Data Synchronization Conventions
Two runtimes:
- Continuous
- Personal periodic

Schould be:
- universal: whether it's continuous or periodic, before generation of recommendations, the state fo available data should be equally synchronized. 
- configurable: the parameters of the synchronization should be externalized from the codebase.


## Questions to consider

1. Should we have RAG? Any rules, policies? Third party analysis and recommendations?
2. Data persistence:
   1. What historical tail of data we need to keep?
   2. If there any aggregations calculated?
   3. Any calculation snapshots? Make sense if the system provides recommendations at the certain timepoint (traceability, auditability).
3. How to handle the data privacy? Even in personal system makes sense to consider.
4. How to handle the data quality? Missing values (mainly), etc.

