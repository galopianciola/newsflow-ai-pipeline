# Project Structure

This document explains the intended folder layout for NewsFlow AI and
why each part exists.

Good structure matters for two reasons:

- it keeps the code easier to maintain
- it makes the repo easier to review on GitHub

The structure below aims for a clean modular monolith in v1. That
means the code is still one project, but responsibilities are split
clearly enough that later extraction into services remains possible.

## Target Layout

```text
api/
  main.py
  schemas.py
dags/
  newsflow_daily.py
docs/
  architecture.md
  learning-roadmap.md
  project-structure.md
newsflow/
  __init__.py
  config.py
  db.py
  extract.py
  transform.py
  load.py
  llm/
    __init__.py
    base.py
    openai_provider.py
    anthropic_provider.py
tests/
  test_db.py
  test_transform.py
  test_search_endpoint.py
```

## Folder Responsibilities

### `newsflow/`

This should hold the shared application logic.

### Why it exists

The ETL code and the API need to share some infrastructure:

- configuration
- database access
- provider abstractions
- data-processing logic

Putting that code in one package avoids duplication and keeps the
business logic independent from the entry points.

### Planned modules

| File | Responsibility |
|---|---|
| `config.py` | Centralized settings loading |
| `db.py` | SQLAlchemy engine, session, and ORM models |
| `extract.py` | Fetch external news data and dedupe it |
| `transform.py` | Summarize and embed article content |
| `load.py` | Persist metadata and upsert vectors |
| `llm/base.py` | Provider interface shared across the system |

### `api/`

This folder should contain the FastAPI entry point and API-layer
schemas.

### Why it is separate from `newsflow/`

The API is not the business logic. It is an interface on top of the
business logic.

Keeping the HTTP layer thin is a good habit because it makes the core
behavior easier to test and easier to reuse.

### Planned files

| File | Responsibility |
|---|---|
| `main.py` | FastAPI app creation, route registration, middleware |
| `schemas.py` | Pydantic request and response models |

### `dags/`

This folder should contain Airflow DAG definitions only.

### Why this matters

Airflow should orchestrate work, not contain the full business logic
inline. The DAG should call functions from `newsflow/`, not duplicate
their implementation.

That separation keeps the project easier to test and easier to reason
about outside the Airflow runtime.

### `docs/`

This folder explains the project to humans.

### Why docs deserve a first-class folder

For a portfolio project, documentation is part of the product.
Reviewers should be able to understand:

- what the system does
- why the architecture makes sense
- how the implementation plan is organized

without reading every source file.

### `tests/`

This folder should contain automated tests.

### Why tests live outside the implementation folders

Keeping tests in one dedicated location makes it easier to see
coverage at a glance and keeps production modules focused on runtime
behavior.

The main purpose of tests here is to verify:

- transformation behavior
- database behavior
- API behavior

## Design Principles Behind This Structure

### Separate runtime concerns from business logic

Entry points such as FastAPI and Airflow should be thin wrappers around
reusable application code.

### Centralize infrastructure boundaries

Configuration, database access, and provider abstractions should live
in predictable places. That reduces confusion and prevents "magic"
imports spread across the codebase.

### Keep v1 simple

This project does not need microservices in the first version.

A modular monolith is the right fit because it:

- keeps development faster
- keeps deployment simpler
- still demonstrates clean boundaries

### Let the structure tell the story

When someone opens the repo, the folder layout should already hint at
the system design:

- `dags/` means scheduled workflows
- `api/` means online request handling
- `newsflow/` means reusable core logic
- `docs/` means the design is explained, not hidden

That is good engineering and good presentation.
