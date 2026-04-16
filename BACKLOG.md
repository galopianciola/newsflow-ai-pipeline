# NewsFlow AI — Backlog

Operational task list for day-to-day development. The README has the
strategic roadmap; this file has the granular work.

## Convention

- `[ ]` = todo
- `[x]` = done
- `[~]` = in progress
- `[?]` = blocked / needs decision
- Tasks are grouped by phase. Finish the current phase before starting
  the next. Ideas that appear mid-sprint go to "Parking lot" — do not
  interrupt the current phase.

---

## v1 — Core pipeline (target: 5 days)

### Day 1 — Infra and extract

- [ ] Create `docker-compose.yml` with services: `postgres`, `airflow`, `api`
- [ ] Create `.env.example` with: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`,
      `NEWSAPI_KEY`, `PINECONE_API_KEY`, `PINECONE_INDEX_NAME`,
      `POSTGRES_*`, `LLM_PROVIDER`
- [ ] Verify Airflow UI loads at `localhost:8080` with admin/admin
- [ ] Create `newsflow/config.py` with pydantic-settings loading from `.env`
- [ ] Create `newsflow/db.py` with SQLAlchemy engine, session, and
      `Article` model (see schema in README)
- [ ] Create initial Alembic migration (or `Base.metadata.create_all`
      for v1, migrations can wait for v2)
- [ ] Implement `newsflow/extract.py`:
  - [ ] `fetch_top_headlines(limit=50)` calling NewsAPI
  - [ ] Dedupe by `url` against existing DB rows
  - [ ] Return list of new article dicts
- [ ] Smoke test: run extract manually, confirm 50 articles land in Postgres

### Day 2 — Transform and load

- [ ] Create `newsflow/llm/base.py` with `LLMProvider` abstract class:
      `summarize(text) -> str`, `embed(text) -> list[float]`
- [ ] Implement `newsflow/llm/openai_provider.py`
- [ ] Implement `newsflow/llm/anthropic_provider.py`
  - Note: Anthropic does not provide embeddings. For Anthropic provider,
    fall back to OpenAI embeddings or use a local model. Decide and
    document.
- [ ] Factory function in `newsflow/llm/__init__.py`:
      `get_provider(name) -> LLMProvider`
- [ ] Implement `newsflow/transform.py`:
  - [ ] `summarize_article(article)` → 3-line summary
  - [ ] `embed_article(article)` → vector
  - [ ] Error handling: retry with exponential backoff, skip failed articles
- [ ] Pinecone setup: create index with 1536 dimensions, cosine metric
- [ ] Implement `newsflow/load.py`:
  - [ ] `upsert_to_pinecone(article_id, vector, metadata)`
  - [ ] `update_article_in_postgres(article_id, summary, pinecone_id)`
- [ ] End-to-end manual test: run extract → transform → load as a
      Python script. Confirm 50 articles have summaries in Postgres and
      vectors in Pinecone.

### Day 3 — API

- [ ] Create `api/main.py` with FastAPI app, CORS, exception handlers
- [ ] Create `api/schemas.py` with Pydantic models for requests/responses
- [ ] Implement `POST /search`:
  - [ ] Embed query, Pinecone query top-K, fetch full records from Postgres
  - [ ] Return results ordered by score
- [ ] Implement `POST /ask` (RAG):
  - [ ] Reuse search logic for retrieval
  - [ ] Build prompt with retrieved articles as context
  - [ ] Call LLM (same provider abstraction), get grounded answer
  - [ ] Return answer + list of source articles
- [ ] Implement `GET /articles/{id}`
- [ ] Implement `GET /health` with DB and Pinecone ping
- [ ] Manual testing via `/docs` Swagger UI with 3 different queries

### Day 4 — Airflow DAG and tests

- [ ] Create `dags/newsflow_daily.py`:
  - [ ] Task `extract_articles` → returns new article IDs via XCom
  - [ ] Task `transform_articles` → summaries + embeddings
  - [ ] Task `load_articles` → Postgres + Pinecone writes
  - [ ] Schedule: `@daily`, `catchup=False`, `max_active_runs=1`
- [ ] Trigger DAG manually from Airflow UI, confirm success
- [ ] Write tests:
  - [ ] `tests/test_transform.py` — test summarize with mocked OpenAI
  - [ ] `tests/test_search_endpoint.py` — test /search with mocked Pinecone
  - [ ] `tests/test_db.py` — test Article model CRUD
- [ ] Run `pytest`, confirm all pass
- [ ] Verify `docker compose down && docker compose up -d` brings
      everything back clean

### Day 5 — Polish and ship

- [ ] Finalize `README.md`:
  - [ ] Verify Quick Start instructions work from scratch
  - [ ] Add GIF/screen recording of `/ask` endpoint in action
  - [ ] Review Design Notes for clarity
- [ ] Add LICENSE file (MIT)
- [ ] Add `.gitignore` (Python, Docker, IDE files, `.env`)
- [ ] Push to public GitHub repo
- [ ] Update advisors with Excel cell changes (OpenAI, Anthropic,
      RAG → 1; Pinecone, Vector DBs → 2 solid)
- [ ] Add repo link to CV

### Decisions pending

- [?] Anthropic embeddings fallback — use OpenAI embeddings even when
      `LLM_PROVIDER=anthropic`, or use sentence-transformers locally?
      Lean toward OpenAI fallback for v1 simplicity.
- [?] NewsAPI rate limit strategy if hit during dev — switch to RSS
      feeds from Reuters/AP? Only decide if it becomes a blocker.

---

## v2 — Production-grade backend

### CI/CD

- [ ] GitHub Actions workflow: lint (ruff), format check (ruff format),
      type check (mypy, optional), test (pytest + coverage)
- [ ] Coverage badge in README
- [ ] Build and push Docker image to GHCR on main branch
- [ ] Branch protection on main

### Observability

- [ ] Add Prometheus service to docker-compose
- [ ] Add Grafana service with pre-provisioned dashboard
- [ ] Instrument FastAPI with prometheus-fastapi-instrumentator
- [ ] Custom metrics: articles ingested/day, LLM token usage, LLM latency,
      pipeline errors, search requests/minute
- [ ] Structured logging with `structlog` across all services
- [ ] Dashboard showing: pipeline health, API latency p50/p95/p99,
      LLM cost estimate

### Testing

- [ ] Raise coverage to 85%+
- [ ] Integration tests with testcontainers (real Postgres, mocked Pinecone)
- [ ] Load test `/search` and `/ask` with locust or k6

### Refactor to services

- [ ] Split into `ingestion-service`, `enrichment-service`, `search-service`
- [ ] Each service has its own Dockerfile and entry point
- [ ] Define communication: REST between services, or message queue
- [ ] Document boundaries in README architecture section

---

## v3 — AWS deployment

- [ ] Terraform modules for: VPC, RDS (Postgres), ECS cluster, ECR,
      S3 bucket, CloudWatch log groups, IAM roles
- [ ] ECS Fargate task definitions for each service
- [ ] Port Airflow to MWAA, or run on ECS with EFS for DAGs
- [ ] Secrets via AWS Secrets Manager
- [ ] CloudWatch alarms on key metrics
- [ ] Write blog post / README section documenting the AWS architecture

---

## v4 — Event-driven ingestion

- [ ] Add Redpanda (Kafka-compatible) to docker-compose
- [ ] Refactor `extract` into a producer publishing to `articles.raw` topic
- [ ] Refactor `transform` into a consumer reading from `articles.raw`,
      publishing to `articles.enriched`
- [ ] Refactor `load` into a consumer reading from `articles.enriched`
- [ ] Idempotent consumers with deduplication by article URL
- [ ] Document at-least-once semantics in README

---

## Parking lot (ideas from mid-sprint, not scheduled)

Capture ideas here instead of interrupting the current phase. Review
between phases to decide what to promote.

- Idea: hybrid search (dense + BM25) with re-ranking
- Idea: chunk long articles before embedding instead of embedding
        title+summary only
- Idea: Spanish language support (ties in with thesis background)
- Idea: entity extraction with spaCy as an extra enrichment step
- Idea: UI with a simple Streamlit or Next.js frontend for demo purposes
- Idea: evaluation harness — a small labeled set to measure retrieval
        quality (recall@5) when changing models or chunking strategies