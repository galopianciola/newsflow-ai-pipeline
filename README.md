# NewsFlow AI: Automated Context & Embedding Pipeline 📰🤖

## Project Overview
An automated ETL pipeline that extracts daily news articles, leverages Generative AI to generate summaries and semantic embeddings, and serves them via a RESTful API for context-aware semantic search. 

This project demonstrates the integration of robust data orchestration with modern LLM capabilities.

## Architecture & Tech Stack
* **Orchestration (ETL):** Apache Airflow
* **Generative AI & NLP:** LangChain, OpenAI API (Summarization & Embeddings)
* **Databases:** * PostgreSQL (Relational metadata storage)
  * Pinecone (Vector Database for semantic search)
* **API Layer:** FastAPI
* **Infrastructure:** Docker & Docker Compose

## Pipeline Workflow (DAG)
1. **Extract:** Fetch top daily articles from public APIs.
2. **Transform:** Clean text, generate 3-line summaries via LLM, and extract core entities. Generate vector embeddings for semantic search.
3. **Load:** Insert structured metadata into PostgreSQL and vector embeddings into Pinecone.
4. **Serve:** A FastAPI microservice that takes a user query, vectorizes it, queries Pinecone for nearest neighbors, and fetches the full article context from PostgreSQL.

## Current Status
🚧 **Work in Progress:** Currently establishing the base Airflow DAGs and configuring the LangChain prompt templates.
