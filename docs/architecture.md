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

Here is the high level architecture diagram - not fine-tuned and verified yet:
```mermaid
graph TD
    %% Presentation and Entry
    User((User))-->|API_Request|BFF[FastAPI_BFF]
    
    subgraph Orchestration[Orchestration Layer]
        BFF-->Router{Task_Router}
        Router-->|Sync|SyncAgent[Sync_Agent]
        Router-->|Recommend|RecAgent[Recommendation_Engine]
        Router-->|Compute|CalcAgent[Calculation_Engine]
    end

    subgraph Agents[Agentic Framework]
        SyncAgent-->|Tool_Use|DB_Ext[(External_Sources)]
        
        RecAgent-->ManagerAgent[Manager_Agent]
        ManagerAgent-->SearchTool[RAG_Search]
        ManagerAgent-->StrategyTool[Strategy_Analyzer]
        
        CalcAgent-->MathTool[Python_Interpreter]
    end

    subgraph Persistence[Persistence Layer]
        SyncAgent-->|Upsert|PG[(Postgres_pgvector)]
        RecAgent-.->|Query|PG
        CalcAgent-->|Snapshots|PG
    end

    %% Feedback Loop
    PG-.->BFF
    RecAgent-->BFF
    BFF-->User
```

## Data Synchronization Conventions
Two runtimes:
- Continuous
- Personal periodic

Schould be:
- universal: whether it's continuous or periodic, before generation of recommendations, the state fo available data should be equally synchronized. 
- configurable: the parameters of the synchronization should be externalized from the codebase.


## Questions To Consider

1. Should we have RAG? Any rules, policies? Third party analysis and recommendations?
2. Data persistence:
   1. What historical tail of data we need to keep?
   2. If there any aggregations calculated?
   3. Any calculation snapshots? Make sense if the system provides recommendations at the certain timepoint (traceability, auditability).
3. How to handle the data privacy? Even in personal system makes sense to consider.
4. How to handle the data quality? Missing values (mainly), etc.

