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


## Refactored Diagram

```mermaid

graph TD
    %% User and Frontend
    User((User)) -->|API Request| FE[JS/Streamlit FE]
    
    %% FastAPI Service
    FE -->|API Calls| FastAPI[FastAPI Backend]
    
    subgraph FastAPI_Service[FastAPI Service]
        FastAPI --> Router{Task_Router}
        
        %% Task Routing
        Router -->|Sync| SyncEndpoint[Sync Endpoint]
        Router -->|Recommend| RecEndpoint[Recommendation Endpoint]
        Router -->|Compute| CalcEndpoint[Calculation Endpoint]
        
        %% Sync Endpoint Implementation
        SyncEndpoint -->|Async DB| AsyncDB[Async DB Driver]
        SyncEndpoint -->|Async HTTP| ExternalAPIs[External APIs]
        SyncEndpoint -->|Async File| AsyncFiles[CSV/PDF Handling]
        
        %% Recommendation Endpoint
        RecEndpoint -->|Async RAG| RAGSearch[RAG Search]
        RecEndpoint -->|Async Strategy| StrategyAnalyzer[Strategy Analyzer]
        
        %% Calculation Endpoint
        CalcEndpoint -->|Async Math| PythonInterpreter[Python Interpreter]
        
        %% Data Persistence
        AsyncDB -->|Upsert/Query| PG[(Postgres/pgvector)]
        RAGSearch -->|Query| PG
        StrategyAnalyzer -->|Snapshots| PG
        PythonInterpreter -->|Snapshots| PG
        
        %% Feedback Loop
        PG -->|Data Sync| FastAPI
    end

    subgraph Configuration[Configuration Layer]
        Config[External Configs] -->|Sync Params| SyncEndpoint
        Config -->|Recommend Params| RecEndpoint
        Config -->|Compute Params| CalcEndpoint
    end

    %% Data Sync Patterns
    subgraph Sync_Patterns[Sync Patterns]
        Continuous[Continuous Sync] -->|Configurable| SyncEndpoint
        Periodic[Periodic Sync] -->|Configurable| SyncEndpoint
    end

    %% Async Handling Considerations
    subgraph Async_Safety[Async Safety Measures]
        AsyncDB -.->|Non-blocking| FastAPI
        ExternalAPIs -.->|Non-blocking| FastAPI
        AsyncFiles -.->|Non-blocking| FastAPI
        FastAPI -->|Async Task Queue| TaskQueue[Task Queue]
    end

    %% Connection
    FastAPI -->|Response| User


```



## Consider FastAPI Potential Problems

Common Event Loop Killers (seen in production):
- Using sync DB drivers inside async endpoints
- Blocking file I/O (PDFs, images, CSV exports)
- CPU‑heavy JSON serialization
- Third‑party SDKs that secretly block
- Poor retry logic with time.sleep()
- Pandas or NumPy inside request handlers
When planning respective tasks in a monolith application based on FastAPI we need to be aware of these problems and avoid them. 


## Questions To Consider

1. Should we have RAG? Any rules, policies? Third party analysis and recommendations?
2. Data persistence:
   1. What historical tail of data we need to keep?
   2. If there any aggregations calculated?
   3. Any calculation snapshots? Make sense if the system provides recommendations at the certain timepoint (traceability, auditability).
3. How to handle the data privacy? Even in personal system makes sense to consider.
4. How to handle the data quality? Missing values (mainly), etc.

