# codeRAG-mcp Architecture (Python Edition)

## 1. Overview

codeRAG-mcp is a standalone, production-grade Model Context Protocol (MCP) server meticulously designed to provide contextual code understanding and retrieval-augmented generation (RAG) capabilities. Built with Python 3.11+, it leverages the `modelcontextprotocol/python-sdk` (specifically `FastMCP`) for core MCP functionalities. The architecture emphasizes modularity, testability, scalability, and seamless integration with clients like Claude Desktop and other AI assistants supporting MCP. This document reflects the system architecture derived from `plan.md` and `guidelines.md`.

## 2. Core Architectural Principles

* **Python-centric:** The entire backend is built on the Python ecosystem, ensuring consistency and leveraging a rich set of libraries.
* **MCP First:** Adherence to the Model Context Protocol is paramount, utilizing the official Python SDK.
* **Modularity & Extensibility:** Components are designed for clear separation of concerns, enabling easier maintenance, testing, and future enhancements (e.g., plugin system, configurable RAG strategies).
* **Asynchronous by Design:** Leveraging Python's `asyncio` for high-performance I/O-bound operations, crucial for responsive MCP interactions.
* **Test-Driven:** Development follows a test-first approach, with comprehensive testing at all levels.
* **Observability:** Built-in support for metrics, logging, and tracing to ensure operational visibility.
* **Security by Design:** Security considerations are integrated throughout the development lifecycle.
* **Configurable RAG Pipelines:** The system supports flexible RAG strategies (e.g., standard, graph-based, re-ranking) managed by a central `RAGOrchestrator` and configured via Pydantic models.
* **Idempotency:** Critical state-changing operations (MCP tools, API endpoints) are designed to be idempotent.

## 3. Core Components

### 3.1. MCP Server Layer (FastMCP-based)

* **Protocol Handling & Transport:**
    * Leverages `modelcontextprotocol.FastMCP` from the Python SDK for handling MCP message lifecycle, serialization, and validation.
    * Supports `stdio` transport (for Claude Desktop) and `Streamable HTTP` transport, as configured and managed by the SDK.
* **Tool & Resource Registry:**
    * MCP Tools (e.g., `search_code`, `index_repository`, `get_context`, graph queries) are defined using `@mcp.tool` decorators from the SDK.
    * MCP Resources (e.g., server status, configuration) are defined using `@mcp.resource` decorators.
    * Input/output schemas for tools and resources are defined using **Pydantic** models, ensuring automatic validation and schema generation compatible with the MCP SDK.
* **Session & Context Management (MCP `Context`):**
    * Utilizes the `Context` object provided by the `FastMCP` SDK for accessing server configuration, reporting progress, and invoking other MCP capabilities within tool/resource implementations.
* **Error Handling:**
    * Centralized error handling maps internal exceptions to MCP standard error codes and messages as defined by the SDK. Custom exceptions inherit from base types.

### 3.2. RAG Core

* **`RAGOrchestrator` (`src/search/orchestrator.py`):**
    * Central component responsible for managing and executing the RAG pipeline based on `RAGPipelineConfig` and `RAGStrategiesConfig` (Pydantic models from `src/config.py`).
    * Dynamically activates/deactivates and sequences various RAG components (e.g., hybrid search, graph RAG, re-ranking) based on configuration and potential query-time overrides.
* **Document Processing (`src/processing/`):**
    * **File Handling (`reader.py`):** Reads and parses various file types, supports `.gitignore`-aware file walks, handles encodings.
    * **Chunking (`chunker.py`):** Implements syntax-aware code chunking using **Tree-sitter** (via Python bindings) for multiple languages. Provides fallback text-based chunking.
    * **Metadata Extraction:** Extracts file path, line numbers, language, Git information, and other relevant metadata for chunks.
    * **Parallel Processing:** Uses `ThreadPoolExecutor` for CPU-bound parsing tasks and `asyncio.Queue` for batching embedding requests.
* **Embedding Engine (`src/embeddings/`):**
    * **Embedding Client Interface:** Abstract Base Class (`EmbeddingClient`) defining the contract for embedding generation.
    * **BGE Client:** Utilizes state-of-the-art BGE embedding models (e.g., via `sentence-transformers` library on CPU/GPU).
    * **Test Client:** Provides deterministic, fast embeddings for development and testing.
    * **Batching & Retries:** Supports batched embedding generation with retry logic.
    * **Caching:** Implements caching for generated embeddings (e.g., content-based keys, LRU policy).
* **Vector Storage (`src/vector_store/`):**
    * **Vector Store Client Interface:** Abstract Base Class (`VectorStoreClient`) for interacting with vector databases.
    * **Qdrant Client:** Primary vector store, using the official `qdrant-client` for Python. Supports collection management, batched upserts (idempotent by ID), and filtered semantic search.
    * **Chroma Client:** Fallback vector store using `chromadb`.
    * **In-Memory Store:** Full-featured in-memory vector store for testing or lightweight deployments.
    * **File-based Store:** Simple file-based persistence for metadata and vectors.
* **Search & Retrieval (`src/search/`):**
    * **Hybrid Search:** Combines semantic (vector) search with keyword search (e.g., `ripgrep` wrapper or a Python-based FTS library).
    * **Result Fusion:** Implements Reciprocal Rank Fusion (RRF) or other strategies to combine results from multiple retrieval sources.
    * **Re-ranking (`reranker.py`):** Optional module (controlled by `RAGOrchestrator`) using cross-encoder models or LLM-based re-ranking to improve relevance of initial search results.
    * **Graph RAG (via `src/graph/` and `RAGOrchestrator`):** Integrates graph-based retrieval using Neo4j to leverage code structure and relationships.
* **Context Assembly & Token Management (`src/context/`):**
    * **`TokenBudgetManager`:** Manages token limits for context provided to LLMs, using `tiktoken` for accurate token counting across different models.
    * **Contextual Scaffolding:** Assembles the final context from retrieved chunks, prioritizing and selecting content based on relevance and token budget.
    * **Semantic Cache:** Caches past questions and their retrieved contexts/answers to speed up responses for similar queries.

### 3.3. Code Graph Engine (`src/graph/`)

* **`CodeGraph` Class:** Core abstraction for representing and interacting with the code graph.
* **Backend Adapters:**
    * **Neo4j Client:** Primary graph database backend, utilizing Cypher queries for complex relationship traversal. Graph updates are designed to be idempotent (e.g., using `MERGE`).
    * **NetworkX Adapter:** In-memory graph representation for lightweight scenarios or initial development.
* **Graph Ingestion:**
    * Integrates with the Document Processing pipeline (AST chunker) to extract code entities (functions, classes, calls, imports, etc.) and relationships.
    * Supports incremental and batch updates to the graph.
* **Graph-Aware Tools:** Provides MCP tools for querying the code graph (e.g., find callers, list dependencies).

### 3.4. LLM Interaction Layer (`src/llm/`)

* **LLM Gateway Client:**
    * Abstract Base Class (`LLMClient`) for interacting with Large Language Models.
    * Concrete implementations for **Claude (Anthropic)**, **OpenAI**, and potentially OSS models.
    * Handles API calls, retry logic, provider selection/fallback, and basic cost tracking.
* **Prompt Engineering:** Manages prompt construction, incorporating retrieved context and user queries.
* **Guardrails:** Implements input sanitization and output validation/filtering to enhance safety and reliability.

### 3.5. Configuration Management (`src/config.py`, `config.yaml`)

* **Pydantic Settings:** Uses Pydantic models for defining and validating application configuration.
* **Loading Sources:** Configuration is loaded from YAML files, environment variables, and potentially Vault for secrets.
* **Hot Reloading:** Supports dynamic reloading of configuration where feasible.
* **`AppConfig`:** Main Pydantic model holding all configurations, including `RAGStrategiesConfig` and `RAGPipelineConfig`.
* **Runtime Modes (`app_mode`):** "lite", "full", "custom" modes influence RAG strategy presets and feature availability, managed through `AppConfig`.

### 3.6. Security & Compliance

* **Authentication & Authorization (as per `plan.md` P1.4, P2.5):**
    * If HTTP transport is used extensively beyond basic health/metrics, JWT-based authentication can be implemented.
    * MCP-level authentication mechanisms explored via SDK hooks if available.
    * Role-Based Access Control (RBAC) for MCP tools and resources.
* **Secrets Management:** Integration with **HashiCorp Vault** for secure storage and retrieval of secrets.
* **Input Validation:** Pydantic models enforce strict validation for all API/MCP tool inputs.
* **Rate Limiting:** Using libraries like `slowapi` with Redis for HTTP endpoints.
* **Audit Logging:** Structured, immutable logging for significant events and MCP tool invocations.
* **Data Protection:** Principles for handling sensitive data, PII redaction if necessary.

### 3.7. Plugin System (`src/plugins/`)

* **Plugin Interface:** Defines an Abstract Base Class or Protocol for plugins.
* **Registry & Discovery:** Mechanisms for discovering, loading, and managing plugins.
* **Sandboxing (Future):** Plans for secure sandboxing of plugins (e.g., WASM-based).
* **Hooks:** Allows plugins to extend functionality at various points in the RAG/MCP pipeline (e.g., `on_pre_index`, `on_query`).

## 4. Technical Stack Summary

| Category              | Technology / Library                                       | Notes                                                                 |
| --------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------- |
| **Language Runtime** | Python 3.11+                                               | Modern, async-focused                                                 |
| **MCP Framework** | `modelcontextprotocol/python-sdk` (`FastMCP`)              | Core for MCP server implementation                                    |
| **API Framework** | FastAPI (via `FastMCP`)                                    | For health, metrics, and potential HTTP MCP transport                 |
| **Data Validation** | Pydantic                                                   | Configuration, API/MCP schemas, data models                         |
| **Async Task Queue** | Celery (Optional)                                          | For background jobs (e.g., large-scale ingestion, analysis)           |
| **Vector Database** | Qdrant (Primary), ChromaDB (Fallback)                      | Python clients (`qdrant-client`, `chromadb`)                          |
| **Graph Database** | Neo4j (Primary), NetworkX (Lightweight)                    | `neo4j` Python driver                                                 |
| **KV Store / Cache** | Redis / Valkey                                             | `redis-py` (caching, rate limiting, semantic cache)                   |
| **Code Parsing** | Tree-sitter (Python bindings)                              | Syntax-aware chunking                                                 |
| **Embedding Models** | BGE (via `sentence-transformers` or similar)               | State-of-the-art code embeddings                                      |
| **LLM SDKs** | `anthropic`, `openai`                                      | Client libraries for interacting with LLMs                            |
| **CLI Framework** | Typer                                                      | For command-line interface utilities                                  |
| **Package Management**| `uv`                                                       | Fast, modern Python package manager                                   |
| **Testing** | `pytest`, `pytest-asyncio`, `pytest-cov`, `hypothesis`     | Unit, integration, property-based testing                           |
| **Linting/Formatting**| `ruff`, `black`                                            | Code quality and consistency                                          |
| **Type Checking** | `mypy`                                                     | Static type analysis                                                  |
| **Observability** |                                                            |                                                                       |
| - Logging             | `structlog`, Python `logging`                              | Structured JSON logging                                               |
| - Metrics             | `prometheus_client` (or `prometheus-fastapi-instrumentator`) | Prometheus metrics exposure                                           |
| - Tracing             | OpenTelemetry Python SDK                                   | Distributed tracing                                                   |
| **Monitoring** | Prometheus, Grafana, Loki                                  | External systems for dashboarding, alerting, log aggregation          |
| **Containerization** | Docker, Docker Compose                                     | Development and production environments                               |
| **CI/CD** | GitHub Actions                                             | Automation for tests, builds, deployments                           |
| **Secrets Management**| HashiCorp Vault (via `hvac` client)                        | Secure secret storage and access                                      |
| **Documentation** | MkDocs (Material theme), OpenAPI (via FastAPI)             | Developer and API documentation                                       |

## 5. Key Interfaces & Data Flow

### 5.1. MCP Interaction (Simplified)

1.  **Client (e.g., Claude Desktop, IDE Plugin)** sends MCP request (e.g., `tool_code` for `search_code`) via `stdio` or HTTP.
2.  **`FastMCP` Server** receives, deserializes, and validates the request using Pydantic schemas defined for the tool.
3.  The corresponding **MCP Tool function** (e.g., `async def search_code(...)`) is invoked.
4.  The tool function interacts with the **`RAGOrchestrator`**.
5.  **`RAGOrchestrator`** executes the configured RAG pipeline:
    * (Optional) Query Preprocessing (e.g., GQE).
    * Parallel retrieval from **Vector Store** (Qdrant), **Keyword Search**, **Graph Database** (Neo4j).
    * Results are fused (RRF).
    * (Optional) Re-ranking.
    * Context assembled by **`TokenBudgetManager`**.
6.  (If AI interaction needed, e.g., `ask_ai` tool) Assembled context + query sent to **LLM Gateway Client**.
7.  LLM response processed.
8.  Result is returned to the **`FastMCP` Server**.
9.  Server serializes and sends the MCP response back to the Client.

### 5.2. Indexing Data Flow (Simplified)

1.  **Trigger:** MCP Tool (`index_repository`) or CLI command.
2.  **File Discovery:** `src/processing/reader.py` walks the repository.
3.  **Document Processing:**
    * Files are read.
    * `src/processing/chunker.py` uses Tree-sitter for AST-based chunking.
    * Metadata is extracted.
4.  **Embedding Generation:**
    * Chunks are sent to `src/embeddings/` (BGE client).
    * Embeddings are generated in batches.
5.  **Vector Store Ingestion:**
    * Chunks + metadata + embeddings are upserted into **Qdrant** (`src/vector_store/`).
6.  **Graph Database Ingestion:**
    * Code structure (functions, calls, etc.) extracted during chunking is used to update **Neo4j** (`src/graph/`).
7.  Indexing progress reported via MCP `Context` or CLI.

## 6. Runtime Modes (`app_mode`)

The server supports different runtime modes defined by the `app_mode` configuration setting in `AppConfig`. These modes control the preset configurations for RAG strategies and feature availability, allowing for trade-offs between resource usage, performance, and comprehensiveness:

* **`lite`:** Minimalist setup, potentially using simpler chunking, fallback VDB, no advanced RAG features like Graph RAG or re-ranking. Optimized for low resource environments.
* **`full`:** Enables all advanced features, including sophisticated AST-based chunking, primary VDB (Qdrant), Graph RAG, re-ranking, and potentially more complex strategies.
* **`custom`:** Allows for fine-grained control over individual RAG strategies and components via detailed settings in `RAGStrategiesConfig` and `RAGPipelineConfig`. The `RAGOrchestrator` will respect these granular settings.

## 7. Scalability, Resilience & Performance

* **Asynchronous Operations:** Maximizes throughput for I/O-bound tasks.
* **Connection Pooling:** For databases (Qdrant, Neo4j, Redis).
* **Caching:** Multiple levels of caching (embeddings, LLM responses, semantic cache, re-ranked results).
* **Background Tasks:** Celery (optional) for offloading heavy, non-real-time tasks.
* **Resource Management:** Careful management of memory and CPU, especially for embedding models and LLM interactions.
* **Circuit Breakers & Retries:** Implemented for external service calls (e.g., `tenacity`, `pybreaker`).
* **Health Checks:** `/healthz` (liveness) and `/status` (readiness) endpoints for monitoring by orchestrators.
* **Graceful Shutdown:** Ensures proper cleanup of resources and completion of in-flight requests.

## 8. Monitoring & Observability

* **Structured Logging:** JSON-formatted logs with correlation IDs (`structlog`).
* **Metrics:** Prometheus-compatible metrics exposed via `/metrics` (request counts, latencies, error rates, RAG pipeline performance, token usage).
* **Distributed Tracing:** OpenTelemetry integration for end-to-end request tracing across components and services.
* **Dashboards & Alerting:** Grafana dashboards visualizing key metrics from Prometheus, with alerting configured via Alertmanager. Loki for centralized log aggregation and querying.

## 9. Development Workflow & CI/CD

* **Local Development:** `docker-compose.dev.yml` for setting up all services. `uv` for package and virtual environment management. Hot reloading for FastAPI where applicable.
* **CI/CD Pipeline (GitHub Actions):**
    * Linting (`ruff`, `black`) and Type Checking (`mypy`).
    * Unit, Integration, and Contract Tests (`pytest`).
    * Security Scans (e.g., `pip-audit` via `uv`, `gitleaks`, `TruffleHog`, Semgrep).
    * Build Docker containers.
    * (Future) Deploy to staging/production environments.

