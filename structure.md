codeRAG-mcp/
├── .github/                    # GitHub specific files
│   ├── ISSUE_TEMPLATE/         # Issue templates
│   │   └── *.md
│   ├── PR_TEMPLATE.md          # Pull Request template
│   ├── release-drafter.yml     # Release Drafter configuration
│   └── workflows/              # GitHub Actions CI/CD workflows
│       ├── ci.yml              # Continuous Integration workflow
│       └── cd.yml              # Continuous Deployment workflow (placeholder)
├── .vscode/                    # VSCode specific settings (optional, for team consistency)
│   └── settings.json
├── asdf/                       # asdf-vm tool versioning
│   └── .tool-versions          # Defines versions for Python, Node.js (for dev tools), etc.
├── dev/                        # Development environment specific files & configurations
│   ├── automation/             # (NEW) Automation scripts and context management for development
│   │   ├── context/            # (NEW) Context management for automated development tasks
│   │   │   ├── project_memory.py       # (NEW) Core context management
│   │   │   ├── context_embedder.py     # (NEW) Vector-based context search
│   │   │   ├── session_manager.py      # (NEW) Session state tracking
│   │   │   ├── context_injector.py     # (NEW) Automatic context loading
│   │   │   └── state_tracker.py        # (NEW) Development progress tracking
│   │   ├── generators/         # (NEW) Scripts for generating development artifacts
│   │   │   └── generate_context_prompt.py # (NEW) Auto-generate context prompts
│   │   └── validators/         # (NEW) Scripts for validating development artifacts
│   │       └── validate_context_integrity.py # (NEW) Ensure context consistency
│   ├── devcontainer.json       # VSCode Dev Container configuration
│   ├── docker-compose.dev.yml  # Docker Compose for development services (Qdrant, Neo4j, Redis etc.)
│   ├── gitleaks.toml           # Gitleaks configuration for secret scanning
│   ├── dashboard.html          # Optional interactive config UI for development
│   ├── docs/                   # Optional development-specific documentation (e.g., VitePress/React)
│   ├── langsmith/              # LangSmith Docker Compose setup (optional)
│   │   └── docker-compose.yml
│   ├── openapi/                # OpenAPI specifications (e.g., exported openapi.json)
│   ├── profiles/               # Performance profiling outputs (e.g., flamegraphs)
│   ├── project_context/        # LLM Agent development context files (as per guidelines.md)
│   ├── project_memory/         # (NEW) Persistent storage for development automation context
│   │   ├── database/           # (NEW) Databases for state and session tracking
│   │   │   ├── project_state.db      # (NEW) SQLite database for project state
│   │   │   └── sessions.db           # (NEW) SQLite database for session history
│   │   ├── embeddings/         # (NEW) Storage for context embeddings
│   │   │   ├── context_vectors.faiss # (NEW) Vector embeddings (e.g., FAISS index)
│   │   │   └── context_metadata.json # (NEW) Metadata for embeddings
│   │   └── snapshots/          # (NEW) Snapshots of project state and progress
│   │       ├── sprint_1_complete.json # (NEW) Sprint completion snapshots
│   │       └── daily_progress.json   # (NEW) Daily progress backups
│   ├── scripts/                # Utility scripts specifically for development tasks
│   │   ├── check_env.py        # Automated sanity check script for dev environment
│   │   ├── quick_start_uv.sh   # (NEW) Ultra-fast 2-minute setup script for uv
│   │   └── start_dashboard.sh  # (NEW) Complete setup and dashboard launcher script
│   ├── sourcegraph/            # (NEW) Sourcegraph local setup (optional)
│   │   └── docker-compose.yml  # (NEW) Docker Compose for Sourcegraph
│   ├── structurizr/            # Structurizr Lite tool local setup/data (if needed, distinct from DSLs in /docs/structurizr)
│   └── trufflehog-config.yaml  # TruffleHog configuration for secret scanning
├── docs/                       # Project documentation (for MkDocs)
│   ├── architecture.md         # System architecture overview
│   ├── plan.md                 # Detailed project plan and task breakdown
│   ├── techstack.md            # Technology stack details
│   ├── getting_started.md      # Onboarding guide for new developers
│   ├── sdk_integration_guidelines.md # Guidelines for SDK customization
│   ├── rag_configuration.md    # Details on RAG strategies and configuration
│   ├── images/                 # Images used in documentation (e.g., C4 diagrams)
│   └── structurizr/            # Structurizr DSL files for C4 architecture diagrams
│       └── *.dsl
├── infra/                      # Infrastructure as Code (IaC) and service configurations
│   ├── grafana/                # Grafana configurations
│   │   ├── dashboards/         # Pre-configured Grafana dashboard JSON files
│   │   └── provisioning/       # Grafana provisioning files (datasources, dashboards)
│   │       ├── datasources/
│   │       └── dashboards/
│   ├── prometheus/             # Prometheus configurations
│   │   ├── prometheus.yml      # Prometheus scrape configurations
│   │   └── alerts.yml          # Prometheus alert rules
│   └── vault/                  # HashiCorp Vault configurations (dev mode)
│       └── policies/           # Vault policies for development
├── scripts/                    # General project scripts (build, analysis, etc.)
│   └── gen-flamegraph.sh       # Script to generate flamegraphs from profiling data
├── src/                        # Source code for the codeRAG-mcp application
│   ├── __init__.py
│   ├── app.py                  # Main FastAPI application setup (if FastMCP doesn't cover all)
│   ├── mcp_server.py           # Core MCP server initialization using FastMCP
│   ├── main.py                 # (NEW) Main application entry point (e.g., for `uv run python src/main.py`)
│   ├── cli.py                  # Typer-based Command Line Interface
│   ├── config.py               # Pydantic models for application configuration
│   ├── errors.py               # Custom error definitions and handlers
│   ├── api/                    # Non-MCP HTTP API endpoints (if any beyond health/metrics)
│   │   ├── __init__.py
│   │   └── models.py           # Pydantic models for HTTP API request/response bodies
│   ├── auth/                   # Authentication and authorization logic
│   │   └── __init__.py
│   ├── context/                # Context management, token budgeting
│   │   ├── __init__.py
│   │   └── token_budget_manager.py
│   ├── embeddings/             # Embedding generation services
│   │   ├── __init__.py
│   │   ├── embedding_client.py # ABC for embedding clients
│   │   ├── bge_client.py       # BGE embedding client
│   │   └── test_client.py      # Test embedding client
│   ├── graph/                  # Code graph engine (Neo4j, NetworkX)
│   │   ├── __init__.py
│   │   ├── graph.py            # CodeGraph class and related logic
│   │   └── neo4j_client.py     # Neo4j adapter
│   ├── llm/                    # LLM interaction layer
│   │   ├── __init__.py
│   │   ├── llm_client.py       # ABC for LLM clients
│   │   ├── claude_client.py
│   │   └── openai_client.py
│   ├── mcp/                    # MCP protocol implementation (core logic, base tool/resource schemas & registration)
│   │   ├── __init__.py
│   │   ├── resources.py        # MCP resource definitions (using SDK decorators)
│   │   ├── tools.py            # MCP tool definitions (using SDK decorators)
│   │   └── transport.py        # (NEW) MCP transport configuration (stdio, HTTP)
│   ├── plugins/                # Plugin system architecture
│   │   ├── __init__.py
│   │   └── examples/           # Example plugins
│   ├── processing/             # Document processing (reading, chunking)
│   │   ├── __init__.py
│   │   ├── chunker.py
│   │   └── reader.py
│   ├── search/                 # Search and retrieval components (hybrid search, RAG orchestration)
│   │   ├── __init__.py
│   │   ├── fusion.py           # Result fusion logic (RRF)
│   │   ├── hybrid_searcher.py
│   │   ├── orchestrator.py     # RAGOrchestrator class
│   │   ├── reranker.py         # Re-ranking module
│   │   └── evaluator.py        # Retrieval evaluator (for CRAG)
│   ├── tools/                  # Specific MCP tool implementations or modules supporting complex tools
│   │   └── __init__.py         # For example, if a tool's logic is too large for src/mcp/tools.py
│   ├── utils/                  # Common utility functions and classes
│   │   └── __init__.py
│   └── vector_store/           # Vector store interaction layer
│       ├── __init__.py
│       ├── vector_store_client.py # ABC for vector store clients
│       ├── qdrant_client.py
│       ├── chroma_client.py
│       ├── file_client.py
│       └── in_memory_client.py
├── tests/                      # Test suites
│   ├── __init__.py
│   ├── conftest.py             # Pytest fixtures and global test configurations
│   ├── contract/               # API contract tests (e.g., Schemathesis)
│   │   └── __init__.py
│   ├── e2e/                    # End-to-end tests
│   │   ├── __init__.py
│   │   └── test_m1_slice.py
│   ├── integration/            # Integration tests
│   │   └── __init__.py
│   ├── load/                   # Load tests (e.g., k6 scripts)
│   │   └── __init__.py
│   └── unit/                   # Unit tests
│       └── __init__.py
├── .dockerignore               # Specifies files to ignore in Docker builds
├── .env.example                # Example environment variables file
├── .gitignore                  # Specifies intentionally untracked files that Git should ignore
├── .pre-commit-config.yaml     # Configuration for pre-commit hooks (Black, Ruff, mypy, etc.)
├── config.yaml                 # (NEW) Application configuration file (loaded by Pydantic in src/config.py)
├── Dockerfile                  # Dockerfile for building the main application container
├── Makefile                    # Makefile with common development and operational commands (uv-based)
├── README.md                   # Main project overview, setup, and usage instructions
├── DEV_README.md               # Internal team guidance for development
├── mkdocs.yml                  # MkDocs configuration file
├── mypy.ini                    # Mypy static type checker configuration (or in pyproject.toml)
├── pyproject.toml              # Python project metadata, dependencies, and tool configurations (PEP 621)
└── uv.lock                     # Deterministic lock file for Python dependencies generated by uv
** ALl files (codes, docs, config etc) generated should be saved in a structured format and Do not keep random files in the project root**
