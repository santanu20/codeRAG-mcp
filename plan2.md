# codeRAG-mcp: AI Agent-Friendly Development Plan (Rev 8 Aligned - Final)

This document outlines an MVP-first, layered development plan for the `codeRAG-mcp` server, derived from `plan.md (Rev 8)`. It's designed for an AI agent solo developer, emphasizing a **test-first, build-test-debug-test-build loop** for all implementation tasks. Each sprint aims to deliver a testable increment of functionality.

**Mandatory instruction:** Mark checklist items as finished only when fully implemented, tested, and debugged.

---

## Sprint 0: Project Genesis & Core Dev Environment

**Goal:** Establish a secure, reproducible, observable, and developer-friendly foundational environment using `uv`, with an early emphasis on DevSecOps. (Derived from P0)
*(Principle: For each setup task, verify its successful completion and expected behavior before proceeding.)*

### 1. Core Development Setup & `uv` Bootstrapping
- [ ] Create Repository Scaffold
    - [ ] Set up `src/`, `tests/`, `scripts/`, `infra/`, `docs/`, `dev/` directories. *(Test: Verify directory creation)*
    - [ ] Create a baseline `README.md`. *(Test: Verify file creation)*
- [ ] Implement Dev Bootstrapping with `uv`
    - [ ] Install and verify `uv` package manager (`curl -LsSf https://astral.sh/uv/install.sh | sh`). *(Test: `uv --version` runs)*
    - [ ] Create comprehensive `Makefile` with `uv`-based workflow.
        - [ ] Define `Makefile` targets for `uv` workflow (setup, install, dev, test, lint, clean). *(Test: Each target runs, `make help` is clear)*
        - [ ] Add dependency checks (Docker, asdf, `uv`). *(Test: Checks correctly identify installed/missing dependencies)*
        - [ ] Add `asdf install`. *(Test: Installs tools defined in `.tool-versions`)*
        - [ ] Add `uv sync` for dependency installation. *(Test: Installs dependencies from `pyproject.toml` into a venv)*
        - [ ] Add `uv venv` for virtual environment management. *(Test: Creates and activates a venv)*
        - [ ] Add `uv lock` for generating/updating lock files. *(Test: Generates `uv.lock`)*
        - [ ] Consider making `Makefile` targets idempotent where applicable (e.g., setup scripts). *(Design Principle)*
    - [ ] Create enhanced startup scripts for `uv`
        - [ ] `start_enhanced_dashboard.sh` - Complete setup and dashboard launcher (placeholder for now, focus on core `uv` setup).
        - [ ] `quick_start_uv.sh` - Ultra-fast 2-minute setup (focus on `uv venv && uv sync`). *(Test: Script successfully sets up a basic dev env)*
        - [ ] Smart fallback handling and dependency verification in scripts. *(Design Principle for scripts)*
- [ ] Pin Tool Versions
    - [ ] Create `asdf/.tool-versions` (Python, Node, Go, etc. - define initial Python version). *(Test: `asdf install` respects this)*
    - [ ] Create PEP 621 compliant `pyproject.toml` with `uv` configuration.
        - [ ] Define `[project.dependencies]` (initially empty or with `modelcontextprotocol`).
        - [ ] Define `[project.optional-dependencies]` for development dependencies (e.g., `pytest`).
        - [ ] Configure `[tool.uv]` section for `uv`-specific settings (if any beyond defaults).
        - [ ] Set `[build-system]` to use `hatchling` for modern builds.
        - [ ] Add `modelcontextprotocol` as dependency in `[project.dependencies]`. *(Test: `uv sync` installs it)*
        - [ ] Pin `modelcontextprotocol` to specific version and track releases. *(Action: Choose version)*
        - [ ] Establish process for dependency updates using `uv sync --upgrade` and `uv lock`. *(Document this process)*
    - [ ] Generate and commit deterministic `uv.lock` file using `uv lock`. *(Test: File is generated and matches `pyproject.toml`)*
- [ ] **UV INTEGRATION Tasks from P0 (relevant to initial setup)**
    - [ ] Updated all documentation (e.g., `README.md`, new `DEV_README.md`) to reflect `uv`-based workflow.
    - [ ] Enhanced `Makefile` help system for `uv` commands.
    - [ ] Created comprehensive onboarding guides for `uv` usage (`docs/getting_started.md` initial draft).

### 2. Basic Code Quality & Security Setup
- [ ] Configure Linters, Formatters, and Hooks (Initial Setup)
    - [ ] Configure Black, Ruff, mypy in `pyproject.toml` / `mypy.ini`. *(Test: Linters/formatters run with `uv run <tool>`)*
    - [ ] Create `.pre-commit-config.yaml`.
        - [ ] Add hooks for Black, Ruff, mypy.
        - [ ] Add hooks for `check-yaml`, `check-json`, `end-of-file-fixer`, etc.
    - [ ] Run `pre-commit install`. *(Test: Hooks run on commit)*
- [ ] Implement Secrets and Credential Scanning (Basic Setup)
    - [ ] Configure `gitleaks` with `/dev/gitleaks.toml` (basic rules).
    - [ ] Add `gitleaks` hook to `.pre-commit-config.yaml`. *(Test: `gitleaks` hook runs on commit)*
- [ ] Define Initial Developer Experience (DX) Guidelines
    - [ ] Establish code contribution & review standards (brief initial version).
    - [ ] Document the debugging process & available tools (brief initial version).
    - [ ] Create a `DEV_README.md` for internal team guidance.
    - [ ] Define standards for writing LLM-focused MCP Tool descriptions/docstrings (placeholder, to be detailed later).
    - [ ] Introduce guidelines for designing idempotent operations and APIs/Tools. *(Critical Design Principle)*
    - [ ] Define strategy for API versioning from the outset (for HTTP APIs and potentially MCP Tool versions). *(Design Principle)*

### 3. Basic CI/CD Skeleton
- [ ] Set Up Basic CI/CD Pipeline Skeleton
    - [ ] Create `.github/workflows/ci.yml` (or equivalent for chosen CI system).
    - [ ] **UV INTEGRATION:** CI stages to use `uv sync` for dependency installation. *(Test: CI installs deps using `uv`)*
    - [ ] Implement stages: Lint -> Test (placeholder) -> Build (placeholder) with `uv` commands. *(Test: Basic CI pipeline runs, linter stage passes)*
    - [ ] Aim for 50-90% faster CI pipeline execution with `uv` (benchmark once tests are added).

### 4. Initial Documentation Setup
- [ ] Set Up Live Documentation Preview (MkDocs)
    - [ ] Set up MkDocs under `/docs/`.
        - [ ] Install MkDocs & Material theme (via `uv add --dev mkdocs mkdocs-material`). *(Test: `uv run mkdocs serve` works)*
        - [ ] Create `mkdocs.yml`.
        - [ ] Add initial `index.md` & `architecture.md` (can be very basic for now).
        - [ ] Add build/serve scripts to `Makefile`. *(Test: `make docs-serve` works)*
- [ ] Create initial `docs/rag_configuration.md` detailing RAG strategies, Pydantic configuration, `app_mode` interaction, and pipeline customization (skeleton, to be expanded significantly later).

---

## Sprint 1: MVP - Minimal Viable RAG CLI

**Goal:** Implement the absolute core RAG functionality (Process -> Embed -> Store (In-Mem) -> Search -> LLM) accessible via a simple CLI. (Primarily from P1 Core RAG)
*(Principle: For each component, write unit tests first, then implement, debug, and ensure tests pass.)*

### 1. Core RAG Components - Foundational Models & Interfaces
- [ ] Define core Pydantic models for RAG
    - [ ] `Chunk` model (`src/processing/models.py` or similar). *(Test: Model validation works)*
    - [ ] Basic `VectorDocument`, `SearchQuery`, `SearchResult` models (`src/vector_store/models.py`). *(Test: Model validation)*
- [ ] Implement `EmbeddingClient` interface (ABC) (`src/embeddings/interface.py`).
    - [ ] Define methods like `embed_documents`, `embed_query`. *(Test: N/A for ABC, but for design)*
- [ ] Implement `VectorStoreClient` interface (ABC) (`src/vector_store/interface.py`).
    - [ ] Define methods like `upsert`, `search`, `create_collection`. *(Test: N/A for ABC, but for design)*

### 2. Document Processing (MVP)
- [ ] Implement file reading/parsing (`src/processing/reader.py`)
    - [ ] Handle single file reading. *(Test: Reads file content correctly)*
    - [ ] Handle encodings & basic binary file skip. *(Test: Handles common encodings, skips binaries gracefully)*
    - [ ] Implement language detection (basic, e.g., by extension, in `src/processing/utils.py`). *(Test: Detects Python, text files)*
- [ ] Create a simple chunking system (`src/processing/chunker.py`)
    - [ ] Implement Text chunker (fallback, e.g., fixed size, overlap). *(Test: Chunks text document as expected)*
    - [ ] Implement `Chunk` Pydantic model usage.
- [ ] Implement metadata extraction (file path, basic lang). *(Test: Metadata captured correctly with chunks)*

### 3. Embeddings Generation (MVP - Local BGE)
- [ ] Implement BGE client (`src/embeddings/bge_client.py`) using `sentence-transformers`.
    - [ ] Adhere to `EmbeddingClient` interface.
    - [ ] Handle model loading & CPU placement (GPU optional for now). *(Test: Embeds text, returns vectors of correct dimension)*
    - [ ] Support batch processing with size limits. *(Test: Batching works, handles limits)*
    - [ ] Implement basic retry logic for model loading if applicable.
- [ ] Implement embeddings caching (Simple in-memory LRU for now).
    - [ ] Content-based cache keys. *(Test: Cache hit/miss works as expected)*
- [ ] Ensure embedding generation is idempotent for the same content. *(Test: Same input yields same embedding consistently)*

### 4. Vector Storage (MVP - In-Memory)
- [ ] Implement In-Memory vector store (`src/vector_store/in_memory_store.py`).
    - [ ] Adhere to `VectorStoreClient` interface.
    - [ ] Collection creation/management. *(Test: Create, list, delete collections)*
    - [ ] Document upsert with dimension checking. (Ensure upsert is idempotent based on document ID). *(Test: Upsert, update, check idempotency)*
    - [ ] Cosine similarity search. *(Test: Search returns relevant documents, scores correctly)*

### 5. LLM Proxy Handling (MVP - Claude/OpenAI via API)
- [ ] Create `LLMClient` ABC (`src/llm/interface.py`). *(Design)*
- [ ] Implement a client for one LLM provider (e.g., Claude using `anthropic` or OpenAI using `openai` in `src/llm/claude_client.py` or `openai_client.py`).
    - [ ] Adhere to `LLMClient` interface.
    - [ ] Basic API call to generate text. *(Test: Can send prompt, get response)*
    - [ ] Implement retry logic (`tenacity`). *(Test: Retries on simulated failures)*
    - [ ] Basic Input Sanitization/Guardrails (e.g., very simple keyword checks or length limits, to be expanded later).
- [ ] Implement `TokenBudgetManager` (basic version in `src/context/token_budget.py`).
    - [ ] Integrate `tiktoken` for one model (e.g., Claude). *(Test: Counts tokens correctly)*
    - [ ] Basic context truncation logic. *(Test: Context fits budget)*
- [ ] Implement basic context assembly (`src/context/context_assembler.py`).
    - [ ] Simple concatenation of top-k chunks. *(Test: Assembles context correctly)*

### 6. Basic RAG Pipeline & CLI
- [ ] Implement a basic RAG pipeline script (`src/rag_pipeline.py` or integrated into `main.py`).
    - [ ] Steps: Load Doc -> Chunk -> Embed Chunks -> Store Chunks -> Embed Query -> Search Chunks -> Assemble Context -> Call LLM. *(Test: E2E flow with a sample doc and query returns an LLM response)*
- [ ] Implement CLI support via **Typer** (`src/main.py`).
    - [ ] Add a command for basic RAG search (e.g., `python src/main.py search "my query" --file path/to/doc.txt`). *(Test: CLI command executes the RAG pipeline)*
    - [ ] Use `uv run python src/main.py ...` for commands.

### 7. Initial Testing Setup
- [ ] Configure **pytest** & `pytest-cov` in `pyproject.toml`. *(Test: `uv run pytest` runs, discovers, and executes tests)*
- [ ] Set up test structure (`tests/unit/`, `tests/integration/`).
- [ ] Write Unit Tests for all MVP components (Processing, Embeddings, In-Memory VDB, LLM Client, RAG pipeline logic, CLI command parsing). *(Continuous Process: Test -> Build -> Debug -> Test)*
- [ ] **UV INTEGRATION:** Use `uv run pytest` for testing.
- [ ] **UV INTEGRATION:** Use `uv add --dev <package>`/`uv remove --dev <package>` for test dependency management.

---

## Sprint 2: Persistent RAG & Basic MCP Server

**Goal:** Introduce persistent vector storage (Qdrant), set up a basic MCP server using `stdio` transport, and implement core `search_code` and `index_repository` MCP tools. (Focus on P1 stability and MCP basics)
*(Principle: Ensure all new components are unit tested and integration tested with existing parts. MCP tools must have schema validation.)*

### 1. Configuration Management with Pydantic
- [ ] Use **Pydantic** for `config.yaml` (`src/config.py`).
    - [ ] Implement settings loading (env vars, file). *(Test: Config loads from file and env vars, precedence works)*
    - [ ] Add validation for core settings (e.g., model names, paths). *(Test: Invalid config raises errors)*
    - [ ] Define core Pydantic models for RAG strategies configuration (skeleton: `RAGStrategiesConfig`, `BaseRAGComponentConfig`, `StandardRAGConfig`). *(Design)*
    - [ ] Define Pydantic model for RAG pipeline configuration (`RAGPipelineConfig`) including global settings like token budgets and top-k. *(Design)*
- [ ] Implement Configuration Schema Validation (Use Pydantic for strict `config.yaml` validation). *(Test: Schema validation is enforced)*
- [ ] Integrate config loading with Vault for production secrets (via agent or `hvac`) - Dev mode Vault setup.
    - [ ] Set up Vault service (dev mode, `/infra/vault/`) in `docker-compose.dev.yml`. *(Test: Vault container runs)*
    - [ ] Basic integration to pull a dummy secret. *(Test: App can fetch secret from Vault)*

### 2. Vector Storage - Qdrant Integration
- [ ] Add Qdrant to `docker-compose.dev.yml`.
    - [ ] Define Qdrant service with volume. *(Test: Qdrant container runs, data persists across restarts)*
- [ ] Implement **Qdrant** client (`src/vector_store/qdrant_client.py`) using `qdrant_client`.
    - [ ] Adhere to `VectorStoreClient` interface.
    - [ ] Implement collection creation/management. *(Test: Create, list, delete Qdrant collections)*
    - [ ] Implement `upsert` with batching & retries. (Leverage Qdrant's inherent idempotency for upserts by ID). *(Test: Upsert, update, batching, retries on simulated failures, idempotency)*
    - [ ] Implement `search` (with filtering & scoring). *(Test: Search, filtering works as expected)*
- [ ] Update RAG pipeline & CLI to use Qdrant as configurable VDB. *(Test: E2E with Qdrant works)*

### 3. Core MCP Server Setup (`stdio`)
- [ ] Implement Server Framework (`src/enhanced_mcp_server.py`, `src/app.py` if using FastAPI alongside)
    - [ ] Set up enhanced MCP server using `modelcontextprotocol.FastMCP`.
    - [ ] Implement `stdio` transport using the SDK's `stdio_transport`. *(Test: Server starts, basic handshake over stdio possible with a test client/script)*
    - [ ] Use MCP `Resource` & `Tool` structure via SDK base classes and decorators.
    - [ ] Implement graceful shutdown & background task cancellation (basic).
- [ ] Implement Error Handling & Logging (Initial MCP Focus)
    - [ ] Create a centralized error handling system (basic, in `src/logging_config.py` or server file).
    - [ ] Define custom exception classes (e.g., `IndexingError`, `SearchError`).
    - [ ] Implement mapping of internal errors to MCP standard error codes/messages. *(Test: Errors are correctly mapped for MCP responses)*
- [ ] Implement `Runtime Config & Telemetry` (CLI commands for server)
    - [ ] Add commands to start server (`uv run python src/main.py serve --transport stdio`). *(Test: Server starts with CLI command)*
    - [ ] Integrate with `uv`-based development workflow.

### 4. Core MCP Tools: `index_repository` & `search_code`
- [ ] Implement `index_repository` as an MCP Tool.
    - [ ] Tool inputs: repository path/ID. *(Test: Schema validation for inputs)*
    - [ ] Logic:
        - [ ] File walking (`.gitignore` aware - use a library or implement). *(Test: Walks directory, respects .gitignore)*
        - [ ] Document processing (reuse from Sprint 1: read, chunk, embed).
        - [ ] Upsert to Qdrant.
    - [ ] Design `index_repository` tool to be idempotent (safe re-indexing with hash-based document IDs or similar). *(Critical Test: Re-indexing same content doesn't duplicate or error, state is consistent)*
    - [ ] Implement basic progress reporting via MCP Context API during indexing operations (e.g., files processed). *(Test: Progress messages sent via MCP)*
- [ ] Implement `search_code` as an MCP Tool.
    - [ ] Tool inputs: query string, optional filters. *(Test: Schema validation for inputs)*
    - [ ] Logic: Embed query -> Search Qdrant -> Assemble Context -> Call LLM (reuse from Sprint 1).
    - [ ] Tool outputs: LLM response, cited sources/chunks. *(Test: Schema validation for outputs)*
- [ ] Use SDK decorators/classes (e.g., `@mcp_server.tool()`) to define Resources and Tools.
- [ ] Implement input validation for all MCP tool parameters *using Pydantic models & SDK validation*. *(Critical Test for all tools)*

### 5. Testing & Validation
- [ ] Write Unit Tests for Qdrant client, Pydantic configs, new MCP tools logic.
- [ ] Write Integration Tests for:
    - [ ] `index_repository` tool (end-to-end: files -> Qdrant).
    - [ ] `search_code` tool (end-to-end: query -> Qdrant -> LLM -> response).
    - [ ] MCP server startup and basic tool calls over `stdio`.
- [ ] Create Automated Sanity Check Script (`/dev/scripts/check_env.py`).
    - [ ] Add checks for Docker, asdf, services (Qdrant health endpoint).
    - [ ] Add check to ensure `stdio` server can launch and respond to a basic MCP handshake *using the SDK*. *(Test: Script runs and all checks pass)*
- [ ] Write Onboarding Instructions (`docs/getting_started.md` update).
    - [ ] Explain how to run the server in `stdio` mode *using SDK methods*.
    - [ ] Describe the dev workflow for MCP tools.
- [ ] Create `docs/sdk_integration_guidelines.md` detailing safe SDK customization boundaries and MCP compliance points (initial draft).

---

## Sprint 3: Enhancing RAG - Hybrid Search & Orchestration

**Goal:** Implement hybrid search (keyword + semantic with RRF), introduce a `RAGOrchestrator` for managing search strategies, and make the RAG pipeline more configurable. (Focus on P1 search and orchestration, P3 Advanced Search Scoring)
*(Principle: Test keyword search, vector search, and fusion independently, then test the orchestrated hybrid search. Iteratively improve sparse retrieval component.)*

### 1. Sparse Retrieval Component
- [ ] **Initial Keyword Search:** Implement `ripgrep`-based keyword search wrapper (`src/search/keyword_search_rg.py`).
    - [ ] Develop the wrapper interface to execute `ripgrep` searches on the codebase (indexed file paths). *(Test: Wrapper calls ripgrep, returns results for known keywords)*
    - [ ] Implement logic to parse `ripgrep` output into a structured format. *(Test: Parsing is accurate)*
    - [ ] Define a basic scoring/ranking mechanism for `ripgrep` results. *(Test: Scoring assigns reasonable ranks)*
- [ ] **Advanced Sparse Retrieval (BM25):** Implement library-based sparse retrieval with BM25 (`src/search/keyword_search_bm25.py`).
    - [ ] Integrate Whoosh or similar lightweight indexing library (install via `uv add whoosh`). *(Test: Library integrated)*
    - [ ] Implement BM25 scoring algorithm with configurable parameters. *(Test: BM25 scoring logic is correct)*
    - [ ] Create inverted index for code chunks (or relevant text documents) with term frequency statistics during the main indexing process. *(Test: Index is built correctly)*
    - [ ] Add BM25-based keyword searcher as an alternative/enhancement to `ripgrep`-based search. *(Test: BM25 search returns relevant, scored results)*
    - [ ] Implement index update mechanism for incremental document changes for the BM25 index. *(Test: Index updates correctly)*
    - [ ] Benchmark BM25 vs `ripgrep` performance and accuracy (qualitatively initially).

### 2. Reciprocal Rank Fusion (RRF)
- [ ] Implement Reciprocal Rank Fusion (RRF) (`src/search/fusion.py`).
    - [ ] Function takes multiple ranked lists of document IDs/chunks and k-value. *(Test: Fuses lists correctly based on RRF formula)*
    - [ ] Ensure RRF module can accept and process ranked lists from both dense (vector) and sparse (keyword/BM25) search components.

### 3. `RAGOrchestrator` Implementation
- [ ] Design and implement `RAGOrchestrator` class (`src/search/orchestrator.py`).
    - [ ] Orchestrate vector & configured sparse search (e.g., `ripgrep` or BM25). *(Test: Calls both searchers based on config)*
    - [ ] Apply RRF fusion. *(Test: Fuses results from both searchers)*
    - [ ] Manages the execution of various RAG components based on `RAGPipelineConfig` and `RAGStrategiesConfig`.
    - [ ] Supports sequencing of standard hybrid search (placeholders for Graph RAG and Re-ranking).
    - [ ] Implement logic within `RAGOrchestrator` to dynamically activate/deactivate RAG components based on their `enabled` status and selected type (e.g., `StandardRAGConfig.sparse_retriever_type = 'bm25'`). *(Test: Strategies can be toggled, sparse retriever type can be selected)*
    - [ ] Implement `_is_strategy_active` method in `RAGOrchestrator`. *(Test: Correctly identifies active strategies)*
    - [ ] `RAGOrchestrator` includes comprehensive pipeline tracing for debugging. *(Test: Logs are informative)*
- [ ] Implement metadata filtering logic.
- [ ] Update `search_code` MCP tool to use `RAGOrchestrator`. *(Test: MCP tool now performs hybrid search as configured)*

### 4. Configuration for Hybrid Search & Orchestration
- [ ] Enhance `RAGStrategiesConfig` in `src/config.py`:
    - [ ] Add fields to `StandardRAGConfig` for enabling/disabling sparse search, vector search, selecting sparse retriever type (`'ripgrep'`, `'bm25'`), and configuring RRF. *(Test: Config changes affect orchestrator behavior)*
- [ ] Enhance `RAGPipelineConfig` in `src/config.py`.
- [ ] Update `docs/rag_configuration.md` with details on configuring hybrid search, sparse retrievers, and the orchestrator.

### 5. Semantic Cache (Initial)
- [ ] Implement semantic cache using vector search (`src/context/semantic_cache.py`).
    - [ ] Store query-response pairs, with query embedding.
    - [ ] On new query, search cache first. If similar query found, return cached response.
    - [ ] Basic in-memory implementation for now. *(Test: Cache hits for similar queries, misses for dissimilar)*

### 6. Testing
- [ ] Unit tests for `ripgrep` wrapper, BM25 module (indexing, searching, scoring), `fusion.py`, and `RAGOrchestrator`.
- [ ] Integration tests for hybrid search flow with both `ripgrep` and BM25 options.
- [ ] Update `search_code` MCP tool tests to cover hybrid search scenarios and sparse retriever selection.
- [ ] Write comprehensive validation suite for M1 Slice (`validate_phase1.py`).
    - [ ] **Target 6/6 tests passing**: Configuration, Embeddings, Vector Store, Processing, Context, Search Orchestration. *(Build this validation script step-by-step)*

---

## Sprint 4: Production Foundations - Logging, Monitoring, Security Basics

**Goal:** Establish foundational production-oriented features: structured logging, basic monitoring with Prometheus/Grafana (dev setup), initial secret management, and core security/code quality tools. (Mixture of P0, P1, P2)
*(Principle: For each tool/service integrated, ensure it's configurable, test its basic functionality, and document its usage.)*

### 1. Comprehensive Logging with `structlog`
- [ ] Configure Python Logging (`src/logging_config.py`)
    - [ ] Set up structured logging (`structlog`). *(Test: Logs are structured (e.g., JSON))*
    - [ ] Create and update extensive, comprehensive and centralize logging debugging and error handling modules for all phases, modules and files saved in a structured way in `/logs`. *(Test: Log files are created in `/logs`, structure is as designed)*
    - [ ] Ensure logs are output as JSON (Ensure `stdio` mode logs go to stderr/file, compatible with MCP logging needs). *(Test: Output format is JSON, stdio logs correctly routed)*
    - [ ] **Define correlation ID strategy for logs across services/components (HTTP and MCP contexts).** Implement basic correlation ID injection and logging. *(Test: Correlation ID appears in logs for a request flow)*
- [ ] Integrate `structlog` with comprehensive logging system in MCP server and RAG components. *(Test: All major operations log useful, structured information)*
- [ ] Implement centralized error tracking/aggregation service integration (e.g., Sentry, GlitchTip) - Placeholder/config only for now, or a simple file-based aggregation.

### 2. Monitoring & Metrics (Prometheus & Grafana - Dev Setup)
- [ ] Add Prometheus & Grafana to `docker-compose.dev.yml`.
    - [ ] Define Prometheus service & config (`/infra/prometheus/prometheus.yml`). *(Test: Prometheus container runs, scrapes itself)*
    - [ ] Define Grafana service & provisioning (`/infra/grafana/provisioning/`). *(Test: Grafana container runs, accessible via browser)*
- [ ] Expose Prometheus Metrics from FastAPI/MCP Server (if using FastAPI for HTTP)
    - [ ] Implement `/metrics` endpoint (using `prometheus-fastapi-instrumentator` or SDK hooks if available for MCP stats). *(Test: `/metrics` endpoint exists and shows some metrics)*
    - [ ] Add basic custom metrics (e.g., request counts, tool calls, RAG pipeline stage latencies). *(Test: Custom metrics appear in Prometheus)*
- [ ] Set Up Grafana Dashboards
    - [ ] Provision Prometheus datasource (`/infra/grafana/provisioning/datasources/`). *(Test: Grafana can connect to Prometheus)*
    - [ ] Provision basic dashboards (`/infra/grafana/dashboards/`) for FastAPI/MCP Server & system (e.g., request rate, error rate, latency). *(Test: Basic dashboard displays data from Prometheus)*

### 3. Health Endpoints (If Exposing HTTP)
- [ ] Implement `/healthz` (liveness - simple 200 OK). *(Test: Endpoint returns 200)*
- [ ] Implement `/status` (readiness - check DBs like Qdrant, services). *(Test: Endpoint returns 200 if Qdrant is up, error otherwise)*
- [ ] Ensure health/status can be exposed as MCP Resources or via standard HTTP if needed (placeholder for MCP resource version).

### 4. Security & Code Quality Enhancements
- [ ] Implement Secrets and Credential Scanning (CI)
    - [ ] Set up a `TruffleHog` CI step with `/dev/trufflehog-config.yaml` (basic config). *(Test: TruffleHog runs in CI, detects a test secret)*
- [ ] Set Up Type-checking
    - [ ] Configure `mypy` for Python in CI (ensure it runs via `uv run mypy .`). *(Test: `mypy` runs in CI, catches type errors)*
- [ ] Set Up Dependency Vulnerability Scanning **UV COMPATIBLE**
    - [ ] Integrate `uv run pip-audit` or security scanning within `uv`-managed environment. *(Test: `pip-audit` runs, identifies known vulnerabilities if any)*
    - [ ] Configure `Snyk`/`Dependabot` (if using GitHub) to recognize `uv` setup and `uv.lock` file (may require manual setup or specific actions). *(Action: Investigate and setup)*
    - [ ] Define a policy for addressing vulnerabilities (severity, timeline) - document this.
- [ ] Define Input/Output Security Principles
    - [ ] Document initial strategy for handling untrusted inputs (files, queries).
    - [ ] Document principles for validating/sanitizing LLM outputs (basic, to be expanded).
    - [ ] **Document principles for Secure Software Development Lifecycle (SSDLC) tailored to this project.**
    - [ ] **Outline initial threat modeling exercises to be conducted (document findings).**

### 5. Docker Enhancements for `uv`
- [ ] **NOTE:** All Docker containers updated to use `uv` for dependency installation.
    - [ ] Optimized container build times with `uv`'s speed (target). *(Test: Docker builds are faster)*
    - [ ] Multi-stage builds leveraging `uv` cache. *(Test: Layer caching works with `uv`)*
    - [ ] Reproducible container builds using `uv.lock`. *(Test: Builds are consistent)*
- [ ] Add `docker compose build` / `pull` with `uv`-based containers to `Makefile`.
- [ ] Add `docker compose up -d` (Review for `stdio` vs. HTTP targets) to `Makefile`.
- [ ] Add health check waits for services in `Makefile` startup sequence. *(Test: `make up` waits for services to be healthy)*

### 6. API & Documentation (HTTP focus, if applicable)
- [ ] Manage API Contract & Docs (if FastAPI is used for HTTP)
    - [ ] Ensure FastAPI serves Swagger UI & ReDoc with Pydantic models (for non-MCP/HTTP APIs). *(Test: `/docs`, `/redoc` work)*
    - [ ] Add `openapi.json` generation to CI/`Makefile`. *(Test: `openapi.json` is generated and valid)*
- [ ] Implement Architecture Visualization (Basic C4)
    - [ ] Set up Structurizr Lite (optional, or use other diagramming tool).
    - [ ] Create C4 diagrams (`/docs/structurizr/*.dsl` or image files in `/docs/images/`).
        - [ ] System Context Diagram.
        - [ ] Container Diagram (Showing MCP Transports, Qdrant, etc.).
    - [ ] Add script to auto-export PNGs to `/docs/images/` (if using Structurizr Lite).

---
## Sprint 5: Code Graph - Initial Implementation (In-Memory)

**Goal:** Implement a basic in-memory code graph using `networkx`, integrate it with AST-based chunking for initial population, and expose basic graph queries via MCP tools. (Focus on P1/P2 Graphiti-inspired features - initial setup)
*(Principle: Test graph node/edge creation, querying, and MCP tool integration separately and then together.)*

### 1. Graph Foundation & In-Memory Backend
- [ ] Start with in-memory/lightweight graph backend (`networkx`). Install `networkx` via `uv add`.
- [ ] Implement `CodeGraph` class (`src/graph/graph.py`).
    - [ ] Wrapper around `networkx` graph object.
    - [ ] Methods for adding/getting nodes and edges. *(Test: Nodes/edges can be added, retrieved, properties set/get)*
    - [ ] Use an adapter-based design (placeholder for Neo4j later - `src/graph/backends/base.py`, `src/graph/backends/networkx_backend.py`). *(Design Principle)*
- [ ] Define core node types (File, Function, Class) as Pydantic models (`src/graph/models.py`). *(Test: Model validation)*
- [ ] Define core edge types (Imports, Calls) as Pydantic models or constants. *(Design)*
- [ ] Support basic metadata (path, line numbers) on nodes/edges. *(Test: Metadata stored and retrieved correctly)*

### 2. AST Chunker Integration for Graph Population
- [ ] Set up Tree-sitter & build/download languages (Python initially).
    - [ ] Add Tree-sitter Python grammar to project. *(Test: Can parse a Python file using Tree-sitter)*
- [ ] Enhance `src/processing/chunker.py`:
    - [ ] Implement base AST chunker (Tree-sitter). *(Test: Chunks code based on AST nodes, e.g., functions, classes)*
    - [ ] Implement Python-specific AST chunker to extract entities like functions, classes, and basic calls/imports. *(Test: Correctly identifies Python constructs)*
- [ ] Integrate with Document Processing (`src/processing/`)
    - [ ] Add hook/callback in `Chunker` or processing pipeline to populate `CodeGraph` instance during chunking/file processing. *(Test: Processing a file populates the graph with nodes/edges)*
    - [ ] Ensure graph entity ingestion is idempotent (e.g., using node IDs derived from path/name/line, check existence before creation). *(Critical Test: Processing same file multiple times doesn't duplicate graph elements)*
- [ ] Support single-file ingestion (basic).

### 3. Basic Graph MCP Tools
- [ ] Expose basic graph query endpoints as MCP Tools *using SDK decorators*:
    - [ ] `get_graph_node_details` (Input: Node ID. Output: Node properties). *(Test: Tool returns correct data for a known node ID; schema validation)*
    - [ ] `get_graph_node_neighbors` (Input: Node ID, optional edge type. Output: List of neighbor nodes). *(Test: Tool returns correct neighbors; schema validation)*
    - [ ] `get_graph_statistics` (Output: Number of nodes, edges). *(Test: Tool returns correct stats; schema validation)*
- [ ] If any tools modify graph state (not planned for these basic queries), ensure they are idempotent.

### 4. Testing
- [ ] Unit tests for `CodeGraph` class (node/edge manipulation, metadata).
- [ ] Unit tests for AST chunker enhancements (entity extraction for Python).
- [ ] Integration tests for graph population pipeline (file -> AST parsing -> graph elements).
- [ ] Integration tests for the new graph MCP tools.

---
## Sprint 6: Advanced RAG Components - Re-ranking

**Goal:** Implement a re-ranking module to improve search result relevance and integrate it into the `RAGOrchestrator` as a configurable step. (Focus on P3 Advanced Re-ranking)
*(Principle: Test re-ranker model loading and scoring independently, then test its integration within the RAG pipeline.)*

### 1. Re-ranker Research & Model Selection
- [ ] Research & Select Re-ranker Model/Approach:
    - [ ] Evaluate Cross-Encoder models (e.g., from `sentence-transformers`) for code relevance. *(Action: Research and select a model, e.g., `ms-marco-MiniLM-L-6-v2` or a code-specific one if available and small enough for initial use).*
    - [ ] (Optional for now) Evaluate LLM-based re-ranking.
    - [ ] Choose the initial approach.

### 2. Re-Ranker Module Implementation
- [ ] Implement Re-Ranker Module (`src/search/reranker.py`):
    - [ ] Create `ReRanker` ABC and a concrete implementation (e.g., `CrossEncoderReRanker`). *(Design & Implement)*
        - [ ] Adhere to `ReRanker` ABC.
        - [ ] Handle model loading (from `sentence-transformers` or Hugging Face Hub). *(Test: Model loads successfully)*
        - [ ] Implement scoring method that takes query and list of `Chunk` objects, returns re-ranked list of `Chunk` objects with new scores. *(Test: Scores documents, re-ranks them based on relevance to query)*
        - [ ] Handle batching for efficiency. *(Test: Batching works)*
        - [ ] Support CPU/GPU execution (configurable, default to CPU).
- [ ] Define `ReRankerConfig` Pydantic model in `src/config.py` (e.g., model name, top_n to re-rank, enabled flag). *(Test: Config model validation)*

### 3. Integration with `RAGOrchestrator`
- [ ] Modify `RAGOrchestrator` (`src/search/orchestrator.py`):
    - [ ] Add a re-ranking step, activated by `ReRankerConfig.enabled` from `RAGStrategiesConfig`. *(Test: Re-ranking step is conditional)*
    - [ ] This step should take the results from the fusion (or previous retrieval stage) and pass them to the `ReRanker`.
    - [ ] Ensure results are correctly passed and re-ordered by the `ReRanker`. *(Test: Orchestrator correctly uses re-ranker and updates results)*
- [ ] Update `search_code` MCP tool and RAG pipeline to reflect re-ranked results if enabled.

### 4. Caching for Re-ranking (Optional for this sprint, can be basic)
- [ ] Add a simple in-memory cache for re-ranked results (query + top N doc IDs before reranking -> re-ranked doc IDs and scores) to reduce latency on repeated queries for the same initial set. *(Test: Cache hits for identical pre-reranked sets)*

### 5. Benchmarking & Evaluation (Initial)
- [ ] Benchmark Latency/Accuracy Impact:
    - [ ] Measure the overhead (latency) of the re-ranker module. *(Action: Perform basic timing)*
    - [ ] Evaluate the improvement in RAG evaluation metrics subjectively or with a small, fixed test set of queries. *(Action: Manual review of before/after for a few queries)*

### 6. Testing
- [ ] Unit tests for `ReRanker` module (model loading, scoring, batching).
- [ ] Unit tests for `RAGOrchestrator` integration of the re-ranking step (conditional execution, data flow).
- [ ] Integration tests for the full RAG pipeline with re-ranking enabled/disabled.
- [ ] Update `docs/rag_configuration.md` with details on how to configure the re-ranker.

---
## Sprint 7: Full MCP Compliance & Advanced Tools

**Goal:** Achieve a higher level of MCP compliance, implement more sophisticated MCP tools, enhance security with RBAC concepts and advanced prompt injection defenses, and ensure robust error handling and schema validation. (Focus on P2 MCP Server Compatibility, P2 Security)
*(Principle: Adhere strictly to MCP SDK patterns. Test all tools for schema compliance, functionality, and error handling. Idempotency for state-changing tools is paramount.)*

### 1. MCP Compliance Verification (Core Focus)
- [ ] **ModelContextProtocol SDK**: Use latest stable SDK and track releases. *(Action: Verify, update if needed)*
- [ ] **Stdio/HTTP Transports**:
    - [ ] Ensure full support for `stdio` (already primary).
    - [ ] Implement `Streamable HTTP` transport *as per SDK implementation* if FastAPI is used. Integrate with `FastMCP`. *(Test: Server runs with HTTP transport, basic MCP calls work via HTTP)*
- [ ] **Tool/Resource Registry**: Use MCP decorators/schemas—no custom registry unless 100% MCP-compliant via SDK. *(Verify: All tools registered via SDK)*
- [ ] **Schema Strictness**: Validate all tool/resource schemas using Pydantic & SDK validation mechanisms. *(Ongoing Test: All tools rigorously tested for schema adherence on input/output)*
- [ ] **Error Handling**: Map internal errors to MCP-standard error responses (see SDK docs). Refine from Sprint 2. *(Test: Various error conditions produce correct MCP error responses)*
    - [ ] Ensure proper MCP error responses for idempotency-related issues (e.g., request already processed, if applicable to any new tools).
- [ ] **Claude Desktop Integration**: Test E2E handshake with Claude Desktop app (both transports, if HTTP added). *(Manual Test once HTTP is stable)*
- [ ] **Performance/Health Endpoints**: `/healthz`, `/metrics`, `/status`—serve via HTTP and/or expose as MCP Resources.
    - [ ] Implement MCP Resources for health/status/metrics. *(Test: MCP resources accessible and return correct data)*
- [ ] **Audit Logging**: Log every MCP call (via SDK hooks or wrappers) with inputs/outputs/errors.
    - [ ] Audit logs should indicate if an operation was completed due to an idempotency key (for future stateful tools).
    - [ ] **Ensure audit logs are tamper-evident and securely stored (e.g., structured logs to a specific, potentially secured file/stream).** *(Design & Implement basic secure logging practices)*
- [ ] **Idempotency Design**: Verify that all state-changing MCP Tools (e.g., `index_repository`) and relevant HTTP APIs are designed and implemented to be idempotent. *(Re-verify and document for existing tools)*

### 2. Enterprise-Grade Tool Registry Enhancements (Conceptual/Metadata)
- [ ] Build upon the SDK's Tool/Resource registration *exclusively*.
- [ ] Add tags, health checks (for tool dependencies), dependency tracking for tools (as metadata in SDK registration or separate internal tracking). *(Design how to represent this, potentially in docstrings or a manifest)*
- [ ] Implement audit logging for tool usage (hooking into SDK execution path or custom middleware if FastMCP allows). *(Refine existing logging for this)*

### 3. Advanced MCP Tools & File System Operations
- [ ] Register All Required Tool Types (Expand list significantly *as SDK Tools*).
    - [ ] File system tools:
        - [ ] `read_file` (Input: path. Output: content). *(Test: Reads file, handles errors like not found; schema)*
        - [ ] `list_files` (Input: path. Output: list of files/dirs). *(Test: Lists directory contents; schema)*
        - [ ] `write_file` (Input: path, content. Output: status).
            - [ ] Ensure `write_file` tool is idempotent (e.g., writing the same content multiple times has the same effect). *(Critical Test: Idempotency)*
            - [ ] **For tools with side effects (e.g., `write_file`), implement robust confirmation/preview mechanisms if appropriate for the MCP context (e.g., a `dry_run` parameter or specific return codes).** *(Design & Implement if feasible for MCP)*
            - *(Security Note: This tool needs strict RBAC/sandboxing in a real production system. For now, focus on functionality and idempotency.)*
- [ ] Code analysis tools (basic, using existing graph/AST capabilities if possible):
    - [ ] `get_definition` (Input: symbol, file path. Output: definition snippet and location - can use AST for this). *(Test: Finds definition of a function/class in a file)*
    - [ ] `find_references` (placeholder - complex, might rely on more advanced static analysis or graph features later).
- [ ] Git tools (using `GitPython` library, install via `uv add GitPython`):
    - [ ] `git_diff` (Input: file, revisions. Output: diff text). *(Test: Shows diff for a tracked file)*
    - [ ] `git_log` (Input: file, count. Output: list of log entries). *(Test: Shows log for a file)*
    - [ ] `get_git_stats` MCP tool for repository insights (e.g. top committers, file changes - basic version). *(Test: Returns some basic stats from git)*

### 4. Security Enhancements
- [ ] **RBAC / Policy Checks for Tool Usage** (Define & Enforce, possibly via SDK hooks or wrappers).
    - [ ] Conceptual design: How would RBAC apply to MCP tools?
    - [ ] Implement a basic "allow/deny" list for tools based on a hypothetical user role (can be hardcoded or simple config for now). Test with a wrapper around tool execution. *(Test: "Denied" tool calls are blocked)*
- [ ] Implement Advanced Prompt Injection Defenses (Contextual checks, model-based validation - initial research and basic implementation).
    - [ ] Review common prompt injection techniques.
    - [ ] Implement basic defenses: e.g., strict instruction boundaries in prompts, input length limits, simple keyword filtering on inputs to LLM. *(Test: Basic malicious prompts are somewhat mitigated or rejected)*
- [ ] **Implement more robust LLM Guardrails: input filtering, output scanning (basic keyword based), topic banning (conceptual).**
- [ ] Implement LLM Output Validation/Sanitization (Especially for tools with side effects like `write_file`).
    - [ ] **Define a framework for LLM output validation (e.g., regex for file paths from LLM, Pydantic models for structured output from LLM).** Implement for one tool. *(Test: LLM output for `write_file` path is validated)*
- [ ] **Security Review**: Document formal threat model for all MCP tool/resource entrypoints. *(Action: Create initial threat model document covering existing tools)*

### 5. Testing
- [ ] Unit and Integration tests for all new MCP tools (`read_file`, `write_file`, `list_files`, git tools, etc.), including schema validation, functionality, error handling, and idempotency where applicable.
- [ ] Tests for RBAC PoC.
- [ ] Tests for basic prompt injection defenses and LLM output validation.
- [ ] Ensure MCP contract tests (if a local mock/validator exists or is created) run regularly. If not, ensure rigorous manual validation against MCP principles.

---
## Sprint 8: Production Code Graph - Neo4j & Graph RAG

**Goal:** Transition the code graph to a persistent Neo4j backend, implement Graph RAG strategies, and integrate this into the `RAGOrchestrator`. (Focus on P2 Graph - Full Impl, P3 Graph RAG)
*(Principle: Test Neo4j adapter thoroughly. Test Graph RAG components (retriever, queries) before integrating into the orchestrator. Ensure graph updates are idempotent.)*

### 1. Neo4j Backend Implementation
- [ ] Add Neo4j to `docker-compose.dev.yml`.
    - [ ] Define Neo4j service with volume & APOC plugin (if needed). *(Test: Neo4j container runs, data persists, accessible via browser/cypher-shell)*
- [ ] Implement **Neo4j** Backend Adapter (`src/graph/backends/neo4j_backend.py`).
    - [ ] Adhere to the `CodeGraph` backend interface defined in Sprint 5.
    - [ ] Use `neo4j` Python driver (install via `uv add neo4j`).
    - [ ] Implement connection pooling & retry logic. *(Test: Handles connection issues gracefully)*
    - [ ] Implement methods for adding/getting nodes and edges using Cypher queries.
        - [ ] Utilize Neo4j's `MERGE` command or similar patterns to ensure graph update operations are idempotent. *(Critical Test: `MERGE` queries correctly create or match nodes/edges, ensuring idempotency)*
- [ ] Update `CodeGraph` class to use the Neo4j adapter, configurable via `config.yaml`. *(Test: `CodeGraph` can switch between `networkx` and `Neo4j` backends)*
- [ ] Implement Neo4j Performance Tuning & Indexing strategy.
    - [ ] Define and create necessary indexes on node/edge properties (e.g., node ID, path, type). *(Test: Indexes are created, query performance improves for indexed fields)*

### 2. Advanced Graph Schema & Ingestion
- [ ] Refine graph schema to capture more detailed code relationships (e.g., explicit call relationships, inheritance, usage). Update Pydantic models in `src/graph/models.py`. *(Design)*
- [ ] Enhance AST chunker/parser (`src/processing/chunker.py`) to extract these detailed relationships for Python (and other planned languages later). *(Test: Parser extracts more detailed relationships accurately)*
- [ ] Update graph population logic to use the refined schema and relationships. Ensure idempotency of `MERGE` operations with new properties. *(Test: Graph populated with richer data; still idempotent)*
- [ ] Implement batch ingestion for large codebases (if processing many files, batch Cypher queries). *(Test: Batching works and improves performance for large imports)*

### 3. Graph RAG Implementation
- [ ] **Design Graph-Based Query Strategies:**
    - [ ] Define Cypher/Graph queries for common code patterns (e.g., "find functions called by X", "show usages of class Y", "find files that import module Z"). *(Design & Test: Cypher queries run directly in Neo4j and return expected results)*
- [ ] **Implement Graph Traversal for Retrieval:**
    - [ ] Create `GraphRetriever` class in `src/search/graph_retriever.py`.
    - [ ] Implement methods to execute graph queries (Cypher templates) and extract relevant nodes/subgraphs. *(Test: `GraphRetriever` executes queries, processes results)*
    - [ ] Implement logic to convert graph results (nodes, paths) into `Chunk`-like structures or text snippets usable for RAG context. *(Test: Conversion to RAG-compatible format is correct)*
- [ ] Define `GraphRAGConfig` Pydantic model in `src/config.py` (e.g., enabled flag, specific query strategies to use). *(Test: Config model validation)*

### 4. Integration with `RAGOrchestrator`
- [ ] **MODIFIED** `Modify `RAGOrchestrator` (`src/search/orchestrator.py`) to include `GraphRetriever` as a component, activated by `GraphRAGConfig.enabled` from `RAGStrategiesConfig`.` *(Test: GraphRetriever is conditionally activated)*
- [ ] **MODIFIED** `Enhance fusion logic in `RAGOrchestrator` (or a dedicated fusion module it calls) to incorporate graph results alongside vector/keyword results. This might involve a new fusion strategy or weighted combination.` *(Test: Graph results are fused with other search results)*
- [ ] Experiment with weighting and prioritization of graph vs. other sources within `RAGOrchestrator`. *(Action: Initial heuristics, configurable if possible)*

### 5. Advanced Graph MCP Tools
- [ ] Enhance Graph API & MCP Integration (Advanced)
    - [ ] Add `query_code_graph` (Input: Cypher query string or predefined query name + params. Output: Graph data/results) as an MCP Tool. (Security: Parameterize queries, don't allow arbitrary Cypher from LLM directly without sanitization/validation). *(Test: Tool executes predefined/safe queries, returns results; schema validation)*
    - [ ] Add `list_related_symbols` (e.g., uses `GraphRetriever` for common patterns like callers/callees) as an MCP Tool. *(Test: Tool returns relevant symbols from graph)*
    - [ ] Design all graph modification tools/resources (if any are added beyond indexing) to be idempotent.

### 6. Testing
- [ ] Unit tests for Neo4j adapter (CRUD operations, idempotency of `MERGE`).
- [ ] Unit tests for enhanced AST parser (relationship extraction).
- [ ] Unit tests for `GraphRetriever` (query execution, result conversion).
- [ ] Integration tests for `RAGOrchestrator` with Graph RAG enabled (data flow, fusion).
- [ ] Integration tests for new/updated graph MCP tools.
- [ ] Update `docs/rag_configuration.md` with details on configuring Graph RAG and Neo4j.

---
## Sprint 9: Scaling & Extensibility - Multi-Repo & Plugins

**Goal:** Implement multi-repository support, allowing indexing and searching across several codebases. Design and implement the core architecture for a plugin system to extend functionality. (Focus on P3 Multi-Repo, P3 Plugin System)
*(Principle: Test multi-repo indexing and search isolation/federation. For plugins, test discovery, loading, and basic hook execution with a sample plugin.)*

### 1. Multi-Repository Support
- [ ] Implement Repository Registry (`src/multi_repo/registry.py`).
    - [ ] Store repository configurations (name, path, potentially VDB collection name). Could be a simple JSON file or part of `config.yaml`.
    - [ ] Add `codemcp repo add/list/remove` CLI commands (via Typer in `src/main.py`) to manage the registry. *(Test: CLI commands correctly modify the registry; operations are idempotent)*
    - [ ] Implement auto-clone & update logic (e.g., using `GitPython` to clone if path doesn't exist, or pull if it does). *(Test: `repo add` can clone, subsequent calls can update)*
- [ ] Update Indexing Process (`index_repository` MCP tool or underlying logic):
    - [ ] Support indexing a specific registered repository or all registered repositories.
    - [ ] Ensure data from different repositories can be distinguished in the VDB (e.g., separate Qdrant collections per repo, or a `repo_id` metadata field if using a single collection). Choose one approach (separate collections preferred for isolation). *(Design & Implement: Test indexing into separate collections/with metadata)*
- [ ] Update Search Process (`RAGOrchestrator`, `search_code` MCP tool):
    - [ ] Allow specifying a target repository/repositories for search, or search across all.
    - [ ] If using separate VDB collections, search needs to query a specific collection or federate results from multiple. *(Test: Search can target specific repo or all, results are correct)*
- [ ] Implement Multi-Repo Context Budgeting (Basic):
    - [ ] Prioritize context from "active" repo (e.g., specified in query or via MCP state) if budget is tight. *(Design & Implement basic prioritization logic)*
- [ ] Expose repository management as MCP Resource/Tools (e.g., `list_repositories`, `add_repository_config`). (Ensure these are idempotent). *(Test: MCP tools for repo management work; schema validation)*

### 2. Plugin System Architecture & Implementation
- [ ] Design Plugin Architecture:
    - [ ] Create `src/plugins/` directory.
    - [ ] Define plugin interface (e.g., an ABC `BasePlugin` in `src/plugins/base.py` with methods like `setup()`, `get_name()`).
    - [ ] Implement plugin registry and discovery (e.g., using `importlib.metadata` entry points, or simple directory scanning). *(Test: Discovers plugins placed in a specific directory or installed as packages)*
    - [ ] Implement plugin loading and initialization. *(Test: Plugins are loaded and their `setup()` method is called)*
- [ ] Implement Plugin Hooks:
    - [ ] Define key hooks in the RAG/MCP pipeline (e.g., `on_pre_index_document(doc)`, `on_pre_rag_query(query_data)`, `on_post_llm_response(response_data)`). Use a simple event dispatcher or callback list for hooks.
    - [ ] Implement hook calling mechanism at relevant points in the codebase. *(Test: Hook methods in sample plugin are called at correct times)*
- [ ] Ensure plugins can register new MCP Tools/Resources *through a controlled SDK gateway*.
    - [ ] Provide a method for plugins to access the `FastMCP` instance or a registration proxy.
    - [ ] Establish guidelines for plugin authors regarding idempotency of their tools if they modify state. *(Documentation)*
- [ ] Implement Plugin Security & Hardening (Initial considerations):
    - [ ] Document security risks of plugins.
    - [ ] For now, plugins run with same permissions as main app. Note future need for sandboxing (WASM, sub-process).
    - [ ] **Define a clear plugin lifecycle management process (install, update, uninstall, disable) - conceptual for now.**

### 3. Sample Plugins
- [ ] Implement a Sample Plugin: Custom Metadata Extractor.
    - [ ] E.g., a plugin that hooks `on_pre_index_document` and adds a specific metadata tag to `Chunk` objects based on file content. *(Test: Sample plugin loads, hook runs, metadata is added)*
- [ ] Implement a Sample Plugin: Query Pre-processor.
    - [ ] E.g., a plugin that hooks `on_pre_rag_query` and appends some text to the user's query. *(Test: Sample plugin modifies query before RAG pipeline)*

### 4. Documentation & Testing
- [ ] Write Plugin Developer Documentation (`docs/plugin_development.md`).
    - [ ] How to create a plugin, available hooks, how to register MCP tools.
    - [ ] Include a section on designing idempotent plugin operations.
- [ ] Unit tests for repository registry and CLI commands.
- [ ] Integration tests for multi-repo indexing and search.
- [ ] Unit tests for plugin discovery, loading, and hook mechanism.
- [ ] Integration tests with sample plugins to ensure they function as expected.
- [ ] Update `docs/rag_configuration.md` for multi-repo setup.

---
## Sprint 10: Language Intelligence - Comprehensive Analysis

**Goal:** Implement a robust language analysis framework using Tree-sitter for various programming languages, enhancing chunking, metadata extraction, and graph population capabilities. (Focus on P3 Language Analysis System)
*(Principle: For each language analyzer, test its parsing accuracy, entity extraction, and integration with chunking and graph building.)*

### 1. Language Analyzer Framework
- [ ] Implement Language Analyzer Framework (`src/analysis/framework.py` or similar).
    - [ ] Design Python ABC for `LanguageAnalyzer` (e.g., methods `analyze_code(code_string, language)`, `get_supported_languages()`).
    - [ ] Implement a manager/registry for available analyzers. Discovers analyzers (e.g., via naming convention or registration). *(Test: Manager finds and can instantiate analyzers)*
    - [ ] Configure active analyzers via `config.yaml` (e.g., list of language IDs to enable). *(Test: Configuration correctly enables/disables analyzers)*
- [ ] Integrate Tree-sitter:
    - [ ] Ensure Tree-sitter library is properly set up.
    - [ ] Manage Tree-sitter grammar files (e.g., submodule, download script). Add grammars for initial set of languages.

### 2. Language-Specific Analyzers (Tree-sitter based)
- [ ] For each language below, implement a `LanguageAnalyzer` subclass:
    - [ ] General Structure:
        - [ ] Load correct Tree-sitter grammar.
        - [ ] Traverse AST to identify key structures (functions, classes, variables, imports, calls, comments, etc.).
        - [ ] Output structured data (e.g., list of Pydantic models representing these entities with their locations).
    - [ ] **Backend Focus (Initial Set):**
        - [ ] Implement Tree-sitter Analyzer for Python (enhance from previous sprints). *(Test: Detailed analysis of Python code, accurate entity extraction)*
        - [ ] Implement Tree-sitter Analyzer for JavaScript/TypeScript. *(Test: Analysis of JS/TS, including ES Modules, CommonJS)*
        - [ ] Implement Tree-sitter Analyzer for Java. *(Test: Analysis of Java, class/method identification)*
        - [ ] Implement Tree-sitter Analyzer for Go. *(Test: Analysis of Go, package/func identification)*
    - [ ] **Frontend Focus (Initial Set):**
        - [ ] Implement Tree-sitter Analyzer for HTML. *(Test: Basic structure analysis)*
        - [ ] Implement Tree-sitter Analyzer for CSS/SCSS/Less. *(Test: Basic rule/selector analysis)*
    - [ ] **Systems/DevOps (Initial Set):**
        - [ ] Implement Tree-sitter/Custom Analyzer for YAML (Generic & K8s if distinct patterns). *(Test: YAML structure analysis)*
        - [ ] Implement Tree-sitter/Custom Analyzer for Dockerfile. *(Test: Dockerfile instruction analysis)*

### 3. Integration with Core Systems
- [ ] **Enhanced AST-Aware Chunker:**
    - [ ] Modify `src/processing/chunker.py` to use the `LanguageAnalyzer` framework.
    - [ ] For supported languages, use the detailed AST analysis to make smarter chunking decisions (e.g., chunk at function/class boundaries, keep related blocks together). *(Test: Chunks are more semantically coherent for supported languages compared to fallback text chunker)*
- [ ] **Code Relationship Detection (Enhance for Graph):**
    - [ ] Ensure analyzers extract relationship data (calls, imports, inheritance) suitable for populating the code graph.
    - [ ] Update graph population logic (`src/graph/graph_builder.py` or similar) to use this richer data from analyzers. Graph population should remain idempotent. *(Test: Code graph becomes more detailed and accurate for supported languages)*
- [ ] Implement cross-repo dependency resolution (initial concept for Python `pyproject.toml` or JS `package.json`).
    - [ ] Analyzers can identify declared dependencies.
    - [ ] (Future work) Graph builder could try to link these across known repositories.

### 4. Testing & Health Checks
- [ ] Unit tests for each `LanguageAnalyzer` subclass using sample code snippets for that language. Verify accurate parsing and entity/relationship extraction.
- [ ] Integration tests for the chunker using different language analyzers to ensure correct chunking behavior.
- [ ] Integration tests for graph population using different language analyzers to ensure graph accuracy.
- [ ] Implement `/health/deep` endpoint (if HTTP is used) or an MCP Resource to check the status of loaded language analyzers and their grammars. *(Test: Health check reports status of analyzers)*
- [ ] Expose metrics on analyzer usage (e.g., files processed per language) via Prometheus.

---
## Sprint 11: Cutting-Edge RAG - Self-RAG, CRAG, LongRAG

**Goal:** Implement advanced RAG strategies like Self-RAG, CRAG, and LongRAG as configurable components within the `RAGOrchestrator`, allowing for more intelligent and adaptive retrieval and generation. (Focus on P4 Advanced RAG Enhancements)
*(Principle: Each advanced RAG strategy should be a modular component, independently testable, and then integrated and tested within the orchestrator. Define clear Pydantic configs for each.)*

### 1. Foundational Pydantic Configurations for Advanced RAG
- [ ] In `src/config.py`, under `RAGStrategiesConfig`, define Pydantic models for each new strategy:
    - [ ] `LongRAGConfig` (e.g., `enabled`, `hierarchical_chunking_params`, `summary_model_name`).
    - [ ] `SelfRAGConfig` (e.g., `enabled`, `reflection_model_name`, `retrieval_gate_threshold`).
    - [ ] `CRAGConfig` (e.g., `enabled`, `retrieval_evaluator_model_name`, `web_search_tool_name` if applicable).
    - [ ] `GenerativeQueryExpansionConfig` (e.g., `enabled`, `expansion_llm_model_name`).
- *(Test: All new config models are well-defined and pass Pydantic validation)*

### 2. LongRAG / Extended Token Retriever Implementation
- [ ] **Research & Design:** (Recap from original plan)
    - [ ] Review and select a LongRAG approach (e.g., RAPTOR-like hierarchical indexing, summarization).
- [ ] **Implementation - Indexing (`src/processing/long_context_processor.py`):**
    - [ ] Implement hierarchical chunking (e.g., small chunks, summary chunks, file-level summaries using an LLM). *(Test: Hierarchical chunks and summaries are generated)*
    - [ ] Modify VDB schema/collections to support hierarchical/layered data (e.g., parent-child links in metadata, separate collections for summaries). Ensure indexing remains idempotent. *(Test: Data stored correctly in VDB)*
    - [ ] Update the main indexing pipeline to incorporate this if `LongRAGConfig.enabled`.
- [ ] **Implementation - Retrieval (`src/search/long_rag_retriever.py`):**
    - [ ] Implement multi-stage retrieval (e.g., broad search on summaries -> focused search on base chunks within relevant large contexts). *(Test: Retrieval logic correctly navigates hierarchy)*
    - [ ] Integrate into `RAGOrchestrator` as a conditional component based on `LongRAGConfig`. *(Test: Orchestrator uses LongRAG when configured)*
- [ ] **Implementation - Synthesis:**
    - [ ] Adapt prompt engineering for long-context inputs. Ensure LLM can effectively utilize the provided long context.

### 3. Self-RAG (Retrieval Gate & Reflection) Implementation
- [ ] **Research & Design:** (Recap from original plan)
    - [ ] Define decision points, select/design reflection model mechanism.
- [ ] **Implementation - Retrieval Gate & Reflection Model (`src/search/self_rag_handler.py`):**
    - [ ] Implement pre-retrieval LLM call (the "gate") to decide *if* retrieval is needed (e.g., based on query nature). Use a smaller, faster LLM if possible. *(Test: Gate correctly decides to retrieve or skip based on test queries)*
    - [ ] Implement post-generation LLM call(s) (the "reflection") to critique/evaluate the generated response against retrieved context (Is it supported? Is it relevant?). *(Test: Reflection provides useful critique)*
    - [ ] Implement logic to refine answer or re-retrieve based on critique (simple loop for now).
- [ ] **Integration & Control:**
    - [ ] Integrate Self-RAG logic into `RAGOrchestrator` (e.g., as a wrapper around standard RAG or a series of conditional steps), activated by `SelfRAGConfig`. *(Test: Orchestrator correctly sequences Self-RAG steps)*
    - [ ] **Explore explainability: Log reasons for Self-RAG decisions.**

### 4. Corrective RAG (CRAG) Implementation
- [ ] **Research & Design:** (Recap from original plan)
    - [ ] Design Retrieval Evaluator, define corrective actions.
- [ ] **Implementation - Retrieval Evaluator (`src/search/crag_evaluator.py`):**
    - [ ] Implement an LLM call or heuristic to evaluate relevance/specificity of initially retrieved documents *before* generation. *(Test: Evaluator scores document relevance for a query)*
- [ ] **Implementation - Corrective Actions:**
    - [ ] If evaluation is poor, trigger corrective actions:
        - [ ] E.g., re-query with modifications (see GQE below).
        - [ ] (Optional) Implement a basic web search tool/plugin *as an MCP Tool* (e.g., using DuckDuckGo Search API via a plugin from Sprint 9) and call it if local retrieval is insufficient. *(Test: Web search tool works if implemented)*
    - [ ] Implement logic to augment/replace context based on corrective actions.
- [ ] **Integration & Control:**
    - [ ] Integrate CRAG flow into `RAGOrchestrator`, activated by `CRAGConfig`. *(Test: Orchestrator uses CRAG components when configured)*
    - [ ] **Log and analyze instances where CRAG intervened.**

### 5. Generative Query Expansion (GQE)
- [ ] Implement GQE component (`src/search/query_expander.py`).
    - [ ] Use an LLM to rephrase or expand the original user query to generate multiple variations. *(Test: LLM generates diverse and relevant query variations)*
    - [ ] Perform retrieval using these expanded queries (and potentially the original).
    - [ ] Fuse results from expanded queries.
- [ ] Integrate GQE into `RAGOrchestrator` as an early step, activated by `GenerativeQueryExpansionConfig`. *(Test: Orchestrator uses GQE when configured)*

### 6. `RAGOrchestrator` Enhancements for Advanced Strategies
- [ ] `RAGOrchestrator` to be enhanced to conditionally activate and sequence these advanced RAG components based on their respective configurations in `RAGStrategiesConfig` and `RAGPipelineConfig`. *(Thorough testing of various combinations of strategies)*
- [ ] Fully implement query-time RAG strategy selection in the `search_code` MCP tool by allowing parameters like `enable_strategies: list[str]`, `disable_strategies: list[str]`, `rag_parameters: dict`. `RAGOrchestrator` must respect these if `RAGPipelineConfig.allow_strategy_override_per_query` is true. *(Test: MCP tool can override strategies per query)*

### 7. MCP Interfaces for Advanced RAG Control
- [ ] Design & Implement MCP interfaces:
    - [ ] Implement MCP Resource (e.g., `/config/rag/strategies`, `/config/rag/pipeline`) to expose current RAG configurations. *(Test: Resource returns current config)*
    - [ ] Implement MCP Tool (e.g., `update_rag_strategy_config`) for authorized runtime modification of RAG configurations. This tool must be idempotent and trigger config hot-reloading if supported (requires careful design for hot-reloading). *(Test: Tool can update config, changes are reflected; idempotency)*

### 8. Testing
- [ ] Unit tests for each new RAG component (LongRAG processor/retriever, Self-RAG handler, CRAG evaluator, GQE module).
- [ ] Integration tests for `RAGOrchestrator` with various combinations of advanced RAG strategies enabled.
- [ ] Tests for MCP tool query-time strategy overrides.
- [ ] Tests for MCP config resources/tools for RAG strategies.
- [ ] Develop specific test cases/datasets that would benefit from each advanced strategy and evaluate their impact.
- [ ] Update `docs/rag_configuration.md` extensively for all new strategies and their configurations.

---
## Sprint 12: Operational Excellence & AI Governance

**Goal:** Enhance system resilience, observability, and implement practices for AI safety, explainability, and continuous improvement including a feedback loop. (Focus on P2 Resilience, P2 Observability, P4 XAI, P4 AI Safety, P4 Feedback Loop)
*(Principle: Each operational or governance feature should be testable and demonstrably improve system robustness, transparency, or safety.)*

### 1. Resilience & Performance Enhancements
- [ ] Implement Circuit Breaker & Resilience (`pybreaker` or similar).
    - [ ] Wrap critical external calls (VDB, LLMs, external graph DB if applicable) in circuit breakers. *(Test: Circuit breaker opens on repeated failures, closes after timeout)*
    - [ ] **Add explicit retry mechanisms with jitter and backoff for all external service calls (refine existing ones).** *(Test: Retries with jitter/backoff work as expected)*
- [ ] Implement Caching & Memory Optimization (Refine existing caches):
    - [ ] Semantic Cache Control (Add TTL, option for Redis LFU/LRU via config, Pruning, Heatmap (logging high-use items), *Version Invalidation* if models change). *(Test: TTL, eviction policies work)*
    - [ ] **Design for cache stampede protection for popular items (e.g., using locks or early re-computation).**
- [ ] Implement Data Versioning & Migration Strategy (For Vector/Graph DBs - Conceptual for now, document strategy).
    - [ ] Consider how data migration scripts can be made idempotent.

### 2. Advanced Observability & Monitoring
- [ ] Implement Distributed Tracing (e.g., OpenTelemetry) across services (if multiple distinct services emerge) and *potentially through MCP calls*.
    - [ ] Configure OpenTelemetry SDK. Instrument FastAPI/MCP calls. Export to a backend (e.g., Jaeger, Grafana Tempo - setup in `docker-compose.dev.yml`). *(Test: Traces appear in backend for a request flow)*
    - [ ] **Ensure trace context propagation across asynchronous boundaries (e.g., if using message queues later).**
    - [ ] Enhance `RAGOrchestrator` to emit detailed traces/spans for each active RAG component in a query, including timing and intermediate result metadata. *(Test: Orchestrator steps are visible in traces)*
- [ ] Implement Granular Cost Tracking & Reporting (Per-provider/model).
    - [ ] **Add LLM token usage tracking per tool/request for cost analysis and budget control (log this information).** *(Test: Token counts logged for LLM calls)*
- [ ] Implement Prompt & Response Logging (Ensure sensitive data is handled/masked if necessary). *(Test: Prompts/responses logged, PII masked if rules defined)*

### 3. Explainable AI (XAI) Techniques for RAG
- [ ] **Research & Design XAI for RAG:**
    - [ ] Explore methods to explain *why* certain context was retrieved and *how* it contributed to the final answer.
    - [ ] Consider surfacing chunk scores, source locations more prominently.
- [ ] **Implementation (Initial Steps):**
    - [ ] Enhance `RAGOrchestrator` to collect more detailed provenance/scoring information during its execution.
    - [ ] Modify MCP tool outputs (e.g., `search_code`) to include more detailed citation/provenance metadata (e.g., which specific chunks contributed to which part of the answer, if feasible, or at least confidence scores of chunks). *(Test: Output includes richer citation/score data)*
- [ ] Implement Inline Citation Tracker (more explicit):
    - [ ] Track which chunks led to which part of the response (potentially via enhanced MCP metadata or a separate field).

### 4. Advanced AI Safety Layers
- [ ] **Research & Design Advanced Safety:**
    - [ ] Explore techniques like Constitutional AI or Self-Critique chains for advanced safety (beyond basic guardrails).
- [ ] **Implementation (Building on Sprint 7):**
    - [ ] Refine/extend input/output filtering based on updated policies.
    - [ ] Implement monitoring for attempts to bypass safety layers (e.g., logging suspicious prompts).
- [ ] **Testing & Evaluation:**
    - [ ] Develop red-teaming strategies (manual initially) to test AI safety measures.

### 5. Feedback Learning Loop
- [ ] Add API/MCP Tool for users to rate/correct answers (e.g., `submit_feedback_on_rag_response`).
    - [ ] Input: original query, response, feedback (e.g., rating, correct answer snippet, identified incorrect chunk).
    - [ ] Ensure feedback submission is idempotent if it involves state change (e.g., using a unique feedback ID). *(Test: Feedback tool accepts data; schema validation; idempotency)*
- [ ] Store feedback (e.g., in a JSON file, separate DB table, or dedicated logging stream). *(Test: Feedback is stored correctly)*
- [ ] **Design system for active learning based on feedback (conceptual): how to use this data to improve models or RAG strategies (e.g., for fine-tuning, eval dataset creation).**

### 6. Testing & Evaluation Framework
- [ ] Set up Automated RAG Evaluation Pipeline (Using Ragas/Evidently or custom scripts).
    - [ ] Define core RAG evaluation metrics (Context Precision/Recall, Faithfulness, Answer Relevance - recap from P0).
    - [ ] Choose/create initial evaluation dataset/questions. *(Test: Evaluation pipeline runs, produces metrics report)*
    - [ ] **Integrate evaluation of Responsible AI metrics (bias, fairness, robustness if applicable) into the RAG eval pipeline (requires specialized datasets/tests).**
- [ ] Implement Formal Experimentation Framework (For A/B testing RAG strategies).
    - [ ] Document how to conduct A/B tests (e.g., using different configs, feature flags).
    - [ ] **Ensure feature flags are integrated with the experimentation framework.**

---

## Sprint 13: Ecosystem Integration - IDE Support (Proof of Concept)

**Goal:** Develop a Proof of Concept (PoC) for IDE integration (e.g., VSCode extension) that can communicate with the `codeRAG-mcp` server. (Focus on P4 IDE Integration - PoC)
*(Principle: Test basic communication between IDE extension and MCP server. Focus on one key interaction, like running a search.)*

### 1. IDE Integration Research & Design
- [ ] Choose one IDE for PoC (e.g., VSCode).
- [ ] Research VSCode extension development basics.
- [ ] Design communication strategy:
    - [ ] Connect to MCP Server (`stdio` or HTTP if available and preferred for extension) *using an MCP client (potentially build a minimal client with SDK or use SDK directly if feasible in extension language e.g. Python child process)*.
    - [ ] Decide on a simple interaction flow (e.g., command palette action -> prompt for query -> call `search_code` MCP tool -> display results in output panel).

### 2. Develop VSCode Extension PoC (Separate Project)
- [ ] Set up basic VSCode extension project.
- [ ] Implement UI for invoking a search (e.g., a command). *(Test: Command appears in VSCode)*
- [ ] Implement logic to spawn/connect to the `codeRAG-mcp` server process (if using `stdio`) or make HTTP calls. Ensure server is running independently. *(Test: Extension can start/connect to server)*
- [ ] Implement a call to the `search_code` MCP tool with a hardcoded or user-input query. *(Test: MCP tool call is made, response received)*
- [ ] Display the results from `search_code` in a simple way (e.g., VSCode output window or information message). *(Test: Results are displayed in IDE)*

### 3. (Optional) Language Server Protocol (LSP) Considerations
- [ ] Research if LSP interface is a better long-term approach.
- [ ] For PoC, direct MCP connection is likely simpler. Note LSP as a future enhancement.

### 4. Testing (Manual for PoC)
- [ ] Manually test the VSCode extension PoC:
    - [ ] Install the extension in VSCode.
    - [ ] Run the command.
    - [ ] Verify communication with the `codeRAG-mcp` server.
    - [ ] Verify results are displayed.
- [ ] Document PoC setup and usage.

---
