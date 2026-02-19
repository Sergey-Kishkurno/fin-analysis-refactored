# Solution Architecture

## High Level Architecture Breakdown

Main sections:

1. Presentation Level (streamlit / JS)

3. BFF Level (FastAPI / orchestration)
3. Synchronization Level (Agent Tools / CrewAI or pydantic.ai)
4. Recommendation Engine (Multi-Agentic / CrewAI or pydantic.ai)
5. Calculation Engine (Agent Tools / CrewAI or pydantic.ai)

7. Persistence (Postgres / pgvector)


## Orchestration and Muti-Agent Architecture



## Data Synchronization Conventions
Two runtimes:
- Continuous
- Personal periodic

Need to have universal conventions though.


