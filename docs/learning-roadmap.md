# NewsFlow AI Learning Roadmap

This document turns the project into a guided learning path.

The idea is simple: do not try to "build the whole system." Build one
layer at a time, and at each layer make sure you understand:

- what problem that layer solves
- which concepts it introduces
- what artifact you should produce in the repo
- how to explain the design choice afterward

Use this roadmap together with
[BACKLOG.md](../BACKLOG.md).

## How To Use This Roadmap

For each stage:

1. read the concept summary
2. implement the smallest working version
3. test it in isolation
4. write down what you learned
5. update the docs so the repo stays truthful

That loop is what turns a coding exercise into a portfolio project.

## Stage 0: Frame the Project Clearly

### What you are learning

- how to define scope
- how to describe an architecture before coding
- how to separate current reality from future plans

### Why it matters

A good engineer does not only write code. A good engineer can explain:

- what the system does
- why it exists
- what is in scope
- what is intentionally postponed

### Deliverables

- strong `README.md`
- architecture document
- prioritized backlog

### Key concepts

- scope management
- architectural communication
- portfolio honesty

## Stage 1: Local Development Environment

### What you are learning

- containers
- service boundaries
- environment variables
- reproducible local setup

### Why it matters

Modern backend systems rarely run as one process. Even in a small
portfolio project, it is useful to practice running separate services
locally.

### What to build

- `docker-compose.yml` with `postgres`, `airflow`, and `api`
- `.env.example` for all required settings
- a repeatable startup flow

### Key concepts

- container image vs container instance
- ports, volumes, and networks
- configuration through environment variables
- dependency ordering between services

### What success looks like

- `docker compose up -d` starts the main services
- Airflow UI loads
- PostgreSQL accepts connections
- the API container starts without import errors

## Stage 2: Configuration and Data Model

### What you are learning

- central configuration management
- relational schema design
- code organization for shared infrastructure

### What to build

- `newsflow/config.py` using `pydantic-settings`
- `newsflow/db.py` with SQLAlchemy setup
- first `Article` model

### Key concepts

- source of truth for configuration
- avoiding duplicated `os.environ` access
- unique constraints and deduplication keys
- session management and database lifecycles

### Important design lesson

Before writing ingestion logic, define what an article record means in
your system. That prevents a lot of accidental complexity later.

## Stage 3: Extraction Layer

### What you are learning

- HTTP integrations
- input validation
- deduplication
- defensive programming against third-party APIs

### What to build

- `fetch_top_headlines(limit=50)`
- deduplication by `url`
- basic logging and failure handling

### Key concepts

- raw external data is not trusted
- API clients need timeouts and error handling
- ingestion should be idempotent where possible

### Questions to be able to answer

- Why did you dedupe by `url` instead of `title`?
- What happens if the news API sends the same article twice?
- What happens if the upstream API rate-limits you?

## Stage 4: Transformation and Enrichment

### What you are learning

- prompt design for summarization
- embeddings as numerical representations of text
- abstraction over external AI providers

### What to build

- `LLMProvider` interface
- provider implementations
- `summarize_article(article)`
- `embed_article(article)`

### Key concepts

- summarization is generation
- embeddings are not generation; they are vectors for similarity
- interfaces reduce coupling
- retries and backoff matter for paid external APIs

### Important design lesson

Do not scatter OpenAI or Anthropic calls across the codebase. Put them
behind a clear boundary so the rest of the system depends on your own
interface, not on a vendor SDK.

## Stage 5: Load Layer

### What you are learning

- persistence design
- keeping relational and vector storage aligned
- practical consistency tradeoffs

### What to build

- PostgreSQL write path
- Pinecone upsert path
- logic for storing the link between both records

### Key concepts

- canonical record vs search index
- partial failure handling
- idempotent writes
- metadata needed for later retrieval

### Questions to be able to answer

- Why keep both PostgreSQL and Pinecone?
- Which system is the source of truth?
- What would you do if the Postgres write succeeds and Pinecone fails?

## Stage 6: API Layer

### What you are learning

- request/response modeling
- query-time embeddings
- retrieval pipelines
- grounded generation

### What to build

- `POST /search`
- `POST /ask`
- `GET /articles/{id}`
- `GET /health`

### Key concepts

- schema design with Pydantic
- top-k retrieval
- response shaping
- difference between search results and RAG answers

### Important design lesson

Search and RAG are related but not identical. Search returns matching
documents. RAG uses matching documents as context for generation.

Being able to explain that clearly is valuable in interviews.

## Stage 7: Airflow Orchestration

### What you are learning

- DAG design
- task dependencies
- retries and scheduling
- operational thinking

### What to build

- `dags/newsflow_daily.py`
- separate extract, transform, and load tasks
- daily schedule with manual trigger support

### Key concepts

- task granularity
- XCom or task outputs
- failure isolation
- why orchestration belongs outside business logic

### What success looks like

- the DAG can be triggered manually
- failed articles do not crash the whole batch
- logs are readable enough to debug one broken run

## Stage 8: Testing and Quality

### What you are learning

- mocking external dependencies
- verifying behavior instead of only happy paths
- quality signals for a public repo

### What to build

- tests for transformation logic
- tests for database CRUD behavior
- tests for `/search` endpoint behavior
- lint and format checks

### Key concepts

- unit test vs integration test
- mocking LLM providers and Pinecone
- regression prevention
- readable failure output

### Portfolio lesson

Tests are not only about correctness. In a public repo, they signal
engineering discipline.

## Stage 9: Documentation and Presentation

### What you are learning

- technical writing
- communicating tradeoffs
- presenting work to recruiters and interviewers

### What to build

- polished README
- up-to-date diagrams
- setup instructions that actually work
- screenshots or a short demo GIF

### Key concepts

- documentation is part of the product
- good repos explain both the "what" and the "why"
- honesty about status builds trust

## What To Practice Explaining Out Loud

By the end of v1, you should be comfortable explaining:

- why Airflow is used instead of a simple cron script
- why PostgreSQL and Pinecone both exist
- how embeddings power semantic search
- how the `/ask` endpoint differs from `/search`
- how you handled deduplication and failures
- what you would improve in v2 if the project had more time

If you can explain those clearly, the repo is doing its job as both a
learning vehicle and a portfolio asset.
