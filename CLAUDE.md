# NewsFlow AI — Context for Claude Code

This file tells Claude Code how to work on this project effectively.
Read this before making changes.

## What this project is

An ETL pipeline + RAG API for news articles. Airflow orchestrates daily
ingestion from NewsAPI, LLMs generate summaries and embeddings, data
lands in PostgreSQL (metadata) and Pinecone (vectors), and a FastAPI
service exposes semantic search and RAG endpoints.

This is a portfolio project. Code quality, clean architecture, and
design clarity matter more than shipping features fast.

## Current phase

Check `BACKLOG.md`. Work on the current phase. Do not start tasks from
future phases unless explicitly asked.

## Architecture rules

- **LLM providers go behind `newsflow/llm/base.py` interface.** Never
  import `openai` or `anthropic` directly from `transform.py` or API
  routes. Always use `get_provider(name)`.
- **Database access via SQLAlchemy sessions from `newsflow/db.py`.**
  No raw SQL unless there is a clear reason, and document the reason.
- **Pinecone access is isolated in `newsflow/load.py` and
  `api/routes.py`.** If Pinecone logic starts leaking, extract a
  `VectorStore` interface (planned for v2).
- **Config via `newsflow/config.py` with pydantic-settings.** No
  `os.environ.get(...)` scattered across the code.
- **Errors in the pipeline are logged per-article and do not fail the
  whole DAG run.** One bad article cannot poison the batch.

## Code style

- Python 3.12, type hints everywhere
- Formatter: `ruff format`
- Linter: `ruff check`
- Prefer `pathlib.Path` over `os.path`
- Prefer `pydantic` models over dicts for structured data
- Docstrings on public functions, not on obvious ones

## Commit style

Conventional commits:
- `feat: add /ask endpoint with RAG`
- `fix: handle NewsAPI rate limit with backoff`
- `refactor: extract LLM provider interface`
- `docs: update README with quick start`
- `test: add coverage for transform module`

Small, focused commits. One logical change per commit.

## Common commands

```bash
# Start everything
docker compose up -d

# Tail API logs
docker compose logs -f api

# Run tests
docker compose exec api pytest

# Trigger the DAG manually
docker compose exec airflow airflow dags trigger newsflow_daily

# Connect to Postgres
docker compose exec postgres psql -U newsflow -d newsflow

# Stop and clean
docker compose down -v
```

## What NOT to do

- Do not add dependencies without asking. Keep `requirements.txt` minimal.
- Do not expand scope mid-phase. If an idea appears, add it to the
  "Parking lot" section of `BACKLOG.md` and keep going.
- Do not add authentication, rate limiting, or other "production
  hardening" in v1. Those are explicitly planned for later phases.
- Do not use LangChain beyond what is already in the project. The
  design decision is to keep it minimal. See README Design Notes.
- Do not add new files at the project root. New code goes in
  `newsflow/`, `api/`, `dags/`, or `tests/`.