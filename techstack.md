# codeRAG-mcp Tech Stack  
**Headless MCP Server – Python Edition**  
_Production-grade, protocol-focused, and highly observable architecture for a state-of-the-art MCP/RAG server, fully compatible with `modelcontextprotocol/python-sdk`. Strictly backend, no UI/dashboard included—use external monitoring tools for observability._  

---

## 1. Backend / Orchestration Layer

| Purpose                | Technology                                  | Notes                                                                                  |
|------------------------|---------------------------------------------|----------------------------------------------------------------------------------------|
| Language Runtime       | Python 3.11+                                | Modern async, robust ecosystem.                                                        |
| MCP Framework          | [`modelcontextprotocol/python-sdk`](https://github.com/modelcontextprotocol/python-sdk) (`FastMCP`) | MCP implementation core; stdio + HTTP transports.                                      |
| API Framework          | FastAPI (via FastMCP)                       | Async, typed, auto-OpenAPI docs, health/metrics endpoints.                             |
| Data Validation/Models | Pydantic                                    | Modern validation, serialization for config, schemas.                                  |
| Async Tasks/Jobs       | Celery (+ RabbitMQ/Redis broker)            | (Optional) For async/background tasks; only if required.                               |
| CLI Framework          | Typer                                       | For robust CLI utilities.                                                              |
| API Contract           | OpenAPI (FastAPI/`FastMCP`)                 | Strong schema, auto-generates docs.                                                    |
| Package Management     | [uv](https://github.com/astral-sh/uv)       | Modern, fast Python package/dependency manager.                                        |

---

## 2. Data Storage & Caching

| Purpose             | Technology                        | Notes                                                                       |
|---------------------|-----------------------------------|-----------------------------------------------------------------------------|
| Vector DB (RAG)     | Qdrant (Primary), ChromaDB (Alt)  | Qdrant: High-perf, OSS, Python client. ChromaDB: fallback/lighter use case. |
| Graph DB            | Neo4j Community, NetworkX         | Neo4j: for advanced code graphs. NetworkX: lightweight/in-memory initial dev.|
| Config Management   | Pydantic Models + YAML/Env Vars   | Type-safe config, env var loading.                                           |
| KV Store / Cache    | Redis / Valkey                    | For cache (embeddings, sessions, semantic), queues, session state, etc.      |

---

## 3. Embedding & NLP

| Purpose                 | Technology                                   | Notes                                                         |
|-------------------------|----------------------------------------------|---------------------------------------------------------------|
| Code Embeddings         | BGE (`sentence-transformers` or similar)     | SOTA OSS models for local embeddings, Python-native.           |
| Code Chunking/Parsing   | Tree-sitter (Python bindings)                | AST-aware, language-agnostic code parsing.                    |
| Token Counting          | tiktoken                                     | For LLM token accounting.                                     |

---

## 4. LLM / Retrieval & Orchestration

| Purpose              | Technology                                    | Notes                                                                   |
|----------------------|-----------------------------------------------|-------------------------------------------------------------------------|
| LLM API Access       | Claude (Anthropic), OpenAI, OSS Models        | Python SDKs: `anthropic`, `openai`, etc.                               |
| RAG Orchestration    | Custom (`RAGOrchestrator` in Python)          | Manages retrieval, fusion, context assembly. Configurable via Pydantic. |
|                      | LangChain, LlamaIndex, Haystack (Components)  | Optional: select utils/components, not full frameworks.                 |

---

## 5. Observability / Metrics / Logging

| Purpose              | Technology                                               | Notes                                                                       |
|----------------------|----------------------------------------------------------|-----------------------------------------------------------------------------|
| Metrics Exposure     | `prometheus_client`, `prometheus-fastapi-instrumentator` | `/metrics` endpoint (OpenMetrics), FastAPI integration.                     |
| Monitoring System    | Prometheus + Grafana                                     | External metrics collection, dashboards, alerting (no custom UI).           |
| Health Checks        | FastAPI/Starlette (`FastMCP`)                            | `/healthz` (liveness), `/status` (readiness), auto-exposed endpoints.       |
| Structured Logging   | structlog, Python `logging`                              | JSON-formatted, correlated logs.                                            |
| Log Aggregation      | Loki (with Grafana)                                      | Central log management (external service).                                  |
| Distributed Tracing  | OpenTelemetry Python SDK                                 | E2E request tracing, external Jaeger/Tempo (if needed).                     |

#### **Production Dashboard Workflow (No Custom UI)**
- **Prometheus** scrapes FastAPI’s `/metrics`.
- **Grafana** connects to Prometheus for dashboards.
- **Dashboards:** All visualizations (request count, latency, RAG stats, health, etc.) via Grafana’s web UI—**not inside MCP**.

#### **Minimal HTTP Endpoint Set (if HTTP enabled)**
- `/metrics` – Prometheus format
- `/healthz` – Liveness/readiness
- `/status` – System/context info (JSON)
- `/docs` & `/openapi.json` – Swagger/OpenAPI (auto-generated)

---

## 6. DevOps / CI/CD / Security

| Purpose               | Technology                          | Notes                                                      |
|-----------------------|-------------------------------------|------------------------------------------------------------|
| Containerization      | Docker, Docker Compose              | Parity for prod/dev, easy deployment.                      |
| Infrastructure as Code| Terraform, Pulumi (optional)        | Cloud-ready, reproducible infra.                           |
| CI/CD                 | GitHub Actions                      | Test, lint, build, deploy, sec scans.                      |
| Secrets Management    | HashiCorp Vault (primary), SOPS     | Secure secrets; SOPS for file encryption (alt).            |
| SBOM / Provenance     | Sigstore, Cosign                    | Supply chain security, artifact signing.                   |
| Dependency Scanning   | uv pip-audit, Snyk/Dependabot       | Vulnerability detection in dependencies.                   |
| Static Code Analysis  | Semgrep                             | Security-first static analysis.                            |

---

## 7. Testing & Quality Assurance

| Purpose                 | Technology                                    | Notes                                                    |
|-------------------------|-----------------------------------------------|----------------------------------------------------------|
| Backend Testing         | pytest (+pytest-asyncio, pytest-cov)          | SOTA, async support, coverage.                           |
| Property-Based Testing  | hypothesis                                    | Automated fuzz/edge testing.                             |
| Type Checking           | mypy                                          | Static type enforcement in CI.                           |
| Linting / Formatting    | ruff, black                                   | Code style & formatting.                                 |
| Contract Testing (API)  | schemathesis (optional)                       | OpenAPI-based contract test.                             |
| RAG Evaluation          | Ragas, Evidently AI (frameworks)              | RAG-specific metrics and QA pipelines.                   |

---

## 8. Documentation

| Purpose           | Technology                        | Notes                                                          |
|-------------------|-----------------------------------|----------------------------------------------------------------|
| API Docs (HTTP)   | OpenAPI (FastAPI auto-generated)  | `/docs` (Swagger UI), `/openapi.json`.                         |
| Developer Docs    | MkDocs (Material theme)           | Project docs, architecture, guides.                            |
|                   | Docusaurus (alternative)          | Use if React/JS-based docs required, else prefer MkDocs.       |

---

## ❌ What You Do NOT Need (For Headless MCP Server)

- No React, Next.js, or SPA frameworks.
- No bundled admin UI or custom dashboards (use Grafana externally).
- No extra data viz libraries inside the server.
- No non-API/MCP endpoints except `/metrics`, `/healthz`, `/status`, `/docs`.

---

## 🔒 Example Prometheus Scrape Config

```yaml
scrape_configs:
  - job_name: 'codeRAG-mcp'
    static_configs:
      - targets: ['your-mcp-server-hostname:8000'] # Adjust as needed
```

## Summary Table

| Layer            | Technologies                                            | OSS?   | Required/Optional             |
| ---------------- | ------------------------------------------------------- | ------ | ----------------------------- |
| Backend Core     | Python 3.11+, modelcontextprotocol/python-sdk (FastMCP) | ✔️     | Required                      |
| API Framework    | FastAPI (via FastMCP)                                   | ✔️     | Required                      |
| Vector DB        | Qdrant (Primary), ChromaDB (Fallback)                   | ✔️     | Required                      |
| Graph DB         | Neo4j, NetworkX                                         | ✔️     | Optional (req. for graph RAG) |
| LLM Access       | Claude, OpenAI, Llama3 (SDKs)                           | ✔️/API | Required                      |
| Embeddings       | BGE (sentence-transformers)                             | ✔️     | Required                      |
| Metrics          | prometheus\_client, prometheus-fastapi-instrumentator   | ✔️     | Required                      |
| Healthchecks     | FastAPI/Starlette (via FastMCP)                         | ✔️     | Required                      |
| Logging          | structlog, Python logging, Loki (external)              | ✔️     | Required                      |
| Monitoring UI    | Grafana (external)                                      | ✔️     | Required (external)           |
| Docs             | OpenAPI (FastAPI), MkDocs                               | ✔️     | Required                      |
| CI/CD            | GitHub Actions                                          | ✔️     | Required                      |
| Containerization | Docker, Docker Compose                                  | ✔️     | Required                      |
| Secrets          | HashiCorp Vault                                         | ✔️     | Recommended                   |
| Package Mgmt     | uv                                                      | ✔️     | Required                      |

