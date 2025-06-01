# FAANG-GRADE GOLDEN ENGINEERING & AI PROJECT GUIDELINES (2025+) - codeRAG-mcp Edition (Rev 2.3 - Enhanced LLM Guidance)

---

* **Mission Critical:** `codeRAG-mcp` server MUST follow Anthropic MCP server architecture, guidelines, structure, configuration, ensuring full MCP ecosystem compatibility and efficient `modelcontextprotocol/python-sdk` use.
* **Never try to take shortcut to test and bug fix. try different approach and out of the box thinking but minimize shortcut or bypass implementations**
## Introduction & Core Philosophy

Mandatory engineering, operational, and AI development standards for all projects, with specific enhancements for `codeRAG-mcp`. As a reference standard, these guidelines ensure `codeRAG-mcp`'s FAANG-level robustness, scalability, security, and maintainability.

**Core Tenets:**

* **Test-First, Edge-Case-Driven Development:** No code ships before exhaustive tests.
* **Zero-Loss Context Handoff:** Paramount for `codeRAG-mcp` (requires defined RPO/RTO, tested).
* **Security by Design:** Integrated at every phase.
* **Observability is Key:** If you can't measure it, you can't manage it.
* **Continuous Improvement:** Standards are living, enforced, and iterated.
* **Enforceability:** Via CI/CD, hooks, linting, reviews, audits. Document exceptions. (Audits need process; automate enforcement).

---

# I. Strictly Developmental Workflow & Process (** All points under this section are developmental workflow/process for an LLM agent **)

### 0. LLM Agent Guidance (Part 0)
* **IF YOU ARE AN LLM OR AUTOMATED AGENT, ADHERE STRICTLY TO THE FOLLOWING:** 
    * **General Workflow & Compliance:** Work phase-by-phase, use approved stack, no hallucination, pass all checks, escalate ambiguity, read glossary, meticulous context handoff, follow Development Workflow Checklist and `plan.md`, create handoffs, verify context persistence.
    * **File System & Project Structure Adherence:** CWD, path confirmation, structure awareness, file manifest pre-generation, incremental file operations, handoff & context (structure focus), development context file storage in `/dev/project_context/`. 
    * **Make sure the contexts are comprehensive, self-contained, non-eroding, persistent across chat sessions in nature and make ensure it survives 100+ chat sessions. The current format is good but needs to be more**

This section outlines guidelines related to *how* the project is developed, managed, and the processes followed by the team. These are not directly part of the live, running core application code but are crucial for its successful creation and maintenance.

## A. Development Environment, Lifecycle & Quality (Process Aspects)

### 1. Development Environment & Setup
* **Python Environment:**
    * **Version**: Python 3.11+ for `codeRAG-mcp`. General MCP: Python 3.8+ (3.10+ preferred).
    * **Virtual Env**: Always use. `uv` recommended (per `plan.md` for `codeRAG-mcp`).
        ```bash
        # Using uv
        uv init your-project && cd your-project && uv venv && source .venv/bin/activate
        uv add "mcp[cli]" # Or project dependencies
        ```
* **Repository Governance:**
    * Define repo strategy; use standard layout (per `plan.md`); organize code logically.
    * Use Gitflow/trunk-based; follow Conventional Commits.
    * PRs: Require reviews, passing CI, issue links.
    * Manage env vars securely (the process of managing, not the runtime injection); keep metadata files updated.
    * Define access; use branch protection, CODEOWNERS.
* **Technology Baseline:**
    * Use approved, supported languages (Python 3.11+ for `codeRAG-mcp` per `plan.md`) (the process of selection and approval).
    * Use recommended frameworks (aligns with `codeRAG-mcp` tech stack in `plan.md`) (the process of recommendation).
    * Adhere to 6-month deprecation policy (the policy itself).
* **Dependency Management (Process):**
    * Lock all dependencies (`uv.lock` per `plan.md`) (the act of locking).
    * Use auto-update/scan tools (the process of using these tools); manually review major updates (the review process).
    * Include only necessary libraries (the decision process); keep `pyproject.toml` lean (the maintenance process).
* **Local Development Testing Tools & Practices:**
    * **`mcp inspector` CLI**: Essential for interactive MCP server testing. Use frequently.
        ```bash
        uv run mcp inspector main.py # Or python -m mcp inspector main.py
        ```
    * **Local Client Script (`mcp-client`)**: Use for early `stdio` integration testing.

### 2. Development Lifecycle & Quality Assurance Process
* **Testing Standards (Process Aspects):**
    * Review failed tests in 24h; archive old tests; maintain plans.
    * Use risk-based testing for critical paths (Hypothesis, mutation, etc.) (the approach/process).
* **Test Data Management:**
    * Policies for production-like (anonymized/synthetic) data. Version evaluation datasets. Securely handle/generate test data (the procedures for handling/generating).
* **CI/CD Standards:**
    * Standard CI/CD stages (Setup, Lint, Test, Build, Scan, Deploy, Post-Deploy Test) per `plan.md`.
    * Pipelines as code; use secret stores, matrix builds.
    * Enforce quality gates (fail fast, coverage, vuln. blocking, manual prod gate).
    * Version artifacts; have auto-rollback scripts (the scripts are dev assets, the plan to use them is process). Notify on pipeline failures.
    * Optimize pipelines (cache, parallel, targeted tests) (the process of optimization).
* **Documentation Process:**
    * Maintain standard docs (README, API Docs, etc.). `plan.md` details MkDocs/API doc generation (the tools and process).
    * Docs as code: build/preview on PRs, measure coverage, link from changes (the workflow).
* **Accessibility Process:**
    * Meet WCAG 2.2 AA (the standard to aim for in the process). Test with axe-core, screen readers (the testing procedure).
* **UX & Design Process:**
    * Ensure consistency, clarity, usability (as part of the design and review process). Adhere to UI standards (the adherence process).
* **i18n/L10N Process:**
    * Design for global use (the design consideration/process). Use standard libraries (the selection process).
* **Change Management & Traceability Process:**
    * Track changes via issues/PRs. Trace commits to origin. LLM changes: save prompt as metadata (the process of saving).
* **Performance & Scalability (Process Aspects):**
    * Regular load tests (k6 per `plan.md`) (the act of performing these tests); alert on deviations (the alerting setup process).
    * Plan capacity (e.g., autoscale 70% CPU, 30% mem headroom) (the planning process).
* **Incident Response Process:**
    * Maintain runbooks, quarterly drills, on-call rosters. Detect, investigate, triage (P0-P4) (the procedures). Contain, Eradicate, Recover (the procedures). Blameless postmortems (48h), RCA, track actions (the follow-up process).
* **Resilience & Self-Healing (Process Aspects):**
    * Regular chaos drills (the act of performing them); use blue/green or rolling deployments (the deployment strategy/process).
* **Disaster Recovery & Data Retention (Process Aspects):**
    * Daily auto-backups (the setup and verification process); quarterly restore drills (the drill procedure).
* **Security & Governance (Process Aspects):**
    * Integrate SAST (Semgrep per `plan.md`), DAST (the integration process); quarterly pentests (the act of conducting them).
    * Weekly automated red team/fuzz tests (per `plan.md`) (the process of running these); block releases on failure (the policy).
* **Dependency Provenance & SBOM (Process):**
    * Generate SBOM (Syft), signed provenance per build/release (the generation process). Track, fingerprint, scan all deps (the process); fail builds on missing/unsigned (the CI rule).
* **AI/LLM Specific Standards (Process Aspects):**
    * Evaluation: Weekly LLM eval harnesses (the process of running evaluations); block deployment if drift >5% (the policy). RAG evaluation setup per `plan.md` (the setup process).
* **Ethical AI Review & Bias Mitigation (Process):**
    * Assess RAG/LLM output bias (the assessment process). Guidelines for sensitive/proprietary code (the guidelines themselves are process-related). Ethical review for new AI features (the review process). Responsible AI per `plan.md` (adherence to these principles is a process).

### 3. Mandatory Workflows & Checklists (Part 3)
* **Mandatory Development Workflow (Part 3.A):**
    * **Build-Test-Debug (Test-First):** Aligns with `plan.md`'s loop.
        1. Write comprehensive, edge-case tests FIRST. (MCP: Consider LLM misuse).
        2. Code ONLY after tests exist and define expected behavior.
        3. Lint, type-check frequently.
        4. Run tests; if fail: STOP, debug, fix root cause.
        5. Repeat: write/fix -> lint -> type-check -> test -> debug until 100% pass.
        6. Refactor for clarity/efficiency (tests passing).
        7. Commit/PR ONLY when tests pass, lint clean, types OK.
* **CI/CD Enforcement:** Lint -> Type Check -> Build -> Unit/Integration/Contract Tests -> Security Scans -> Coverage -> Docs Build -> Staging Deploy (if appl.) -> E2E -> Healthcheck (reflects `plan.md` CI) (the enforced sequence).
* **Project Workflow:** Start phase/feature with reqs & test plan. Code to satisfy reqs/pass tests. All auto checks must pass pre-merge. Pass quality gates before phase exit. No skipping checks without approved justification.
* **Pull Request / Feature Checklist (Part 3.B):** (The checklist as a tool for the development process)
    * **General:**
        * [ ] PR: Clear "why" (issue link) & "what" (summary).
        * [ ] Versioned, contract-tested API/interface (if appl.) (the check that this was done). MCP: Ensure compat. (the check).
        * [ ] Tests: Unit/integration/E2E for new logic, edge cases (the check that they exist).
        * [ ] Chaos/abuse tests considered for critical parts (the consideration process).
        * [ ] Security/privacy/compliance reviewed & addressed (the review process).
        * [ ] Docs (comments, docstrings, user/API docs, `CHANGELOG.md`) updated (the check that they are updated).
        * [ ] Peer review (≥1 engineer; 2 for critical); reviewer checklist filled (the review process).
        * [ ] One-click deploy/test/rollback confirmed (if appl.) (the confirmation process).
        * [ ] Metrics/observability updated or sufficient (the check/assessment).
        * [ ] User impact assessed (docs, migration guides) (the assessment process).
        * [ ] LLM/automation PRs: Follow all above; full prompt/context traceability (the adherence check).
    * **`codeRAG-mcp` & MCP Server Specific (Checklist items as process validation):**
        * [ ] Preserves zero-loss graph/context handoff principles (the verification step).
        * [ ] Graph/hybrid search logic tested (the check that tests were run).
        * [ ] Temporal/versioned context principles enforced where applicable (the verification step).
        * [ ] **MCP compat. validated**: Tools/resources/prompts defined via `@mcp.*`. Clear LLM-friendly tool docstrings. Accurate type hints. Robust error handling. Async used correctly. `Context` obj used. Tested with `mcp inspector`. (Opt: local client script test) (all these are verification steps in the PR process).
        * [ ] API, plugin, backend adapters tested (the check that tests were run).
        * [ ] Context/token optimization benchmarks updated (the check that benchmarks are updated).
        * [ ] MCP interaction observability present (the check that it's implemented).
        * [ ] Security (input sanitization, tool permissions) & audit hooks validated (the validation process).
        * [ ] User/LLM feedback, error, fallback handling tested (the check that tests were run).
        * [ ] Idempotency addressed for stateful operations (per `plan.md`) (the verification step).
---

# II. Core Project Implementation

This section outlines guidelines that directly impact the design, architecture, features, and runtime behavior of the `codeRAG-mcp` application itself.

## A. General Engineering Standards for Core Implementation

### 1. Coding Standards & Code Quality (In-Code Implementation)
* Adhere to language guidelines (PEP8/Black, ESLint/Prettier, etc.) in written code. Use standard naming in code.
* Target ≤300 LOC/file, function complexity ≤10 in the implemented code.
* **CRITICAL**: Comprehensive docstrings (Google style/Python) *written in the code files*. MCP tool docstrings: detail purpose, args, returns, side-effects for LLM use (per `plan.md`) *as part of the tool's code definition*.
* Handle errors explicitly, gracefully *in the application logic*; return informative messages *from the application*.
* Use static analysis (Ruff, MyPy), type safety, prefer immutability *in the implemented code and data structures*.
* Apply design patterns, DI, modularity *in the code's architecture*; use atomic commits (commits relate to code implementation).
* Validate inputs, sanitize outputs (prevent injection) *as part of the application's request handling and data processing logic*. Write clean, maintainable, commented code.

### 2. Testing (Implemented Artifacts - Test Code & Coverage)
* Implement comprehensive tests: Unit, Integration, E2E, Performance, Security tests *as runnable code artifacts accompanying the main application code*.
* **Coverage**: Achieve ≥80% unit, ≥70% integration (general) for the implemented code; `codeRAG-mcp`: ≥90% for core/critical code, enforced always. <1% flake rate (per `plan.md` target) for the test suite.
* Use fixtures, mocks, seeding for isolated, reproducible tests *within the test codebase*.

### 3. API Design & Versioning (Implemented in the Server)
* Use consistent API practices (REST/GraphQL, URI versioning) *in the server's implemented endpoints*. MCP: Adhere to protocol & `plan.md` versioning strategies *in the MCP implementation*.
* Enforce API contracts *through schema validation in the server code*; version interfaces *as implemented in the server*; plan for breaks.
* Breaking changes: Require compat. tests, warnings, migration/rollback plan (migration/rollback plan involves implemented code/scripts).
* MCP servers: Use semantic versioning (e.g., `FastMCP("MyApp", version="1.0.0")`) *in the server initialization code*.

### 4. Operations, Reliability & Performance (Implemented Features)
* **Observability & Monitoring (Implemented in Application):**
    * Use structured JSON logs (correlation IDs, 30-day retention) *generated by the application code* per `plan.md` logging setup.
    * Expose Prometheus metrics (`/metrics`, standard labels, SLO alerts) *via an endpoint implemented in the application* per `plan.md` Prometheus setup.
    * Implement OpenTelemetry tracing (configurable sampling) *by integrating the OpenTelemetry SDK into the application code*.
    * Maintain Grafana dashboards linked to runbooks (per `plan.md` provisioning) (dashboards consume implemented metrics).
    * Implement live metrics (usage, perf, errors, cost) with custom alerts *exposed or generated by the application*.
* **Performance & Scalability (Implemented in Application):**
    * Define SLIs/SLOs (e.g., p95 ≤250ms, <1% errors, ≥99.9% avail.) that the *application is designed and implemented to meet*.
    * **Async Everywhere**: All I/O-bound ops *in the application code* must be async (`async`/`await`). Critical for `FastMCP` responsiveness (per `plan.md`).
    * **Resource Caching**: Implement caching *logic within the application* for expensive, infrequently changed data (per `plan.md` caching tasks).
    * **Minimize Blocking**: Avoid long CPU-bound tasks in async loop; use separate process/thread pool (`asyncio.to_thread`) *in the application's async management code*.
    * **Efficient Data Handling**: Implement logic to be mindful of MCP message data size *in the application's data processing*.
* **Immutable Audit Logging (Implemented Feature):**
    * Log state/config changes immutably (append-only, offsite, secure, searchable) per `plan.md` MCP compliance *by the application's logging system*. Include redaction/audit tools (if these are app features).
* **Resilience & Self-Healing (Implemented Features):**
    * Implement health/readiness/liveness endpoints with real checks *within the application* (per `plan.md`).
    * Ensure auto-recovery from crashes, resume from checkpoints *through application logic and state management*.
* **Resource & Abuse Protection (Implemented Features):**
    * Set hard resource limits (if enforced by the application itself).
    * Implement rate limiting (per `plan.md`), abuse detection, block/allow lists *within the application's request handling logic*.
* **Disaster Recovery & Data Retention (Implemented Features):**
    * Clear data retention/auto-expiry *logic implemented within the application if it manages its own data lifecycle*. User/project data export/import tools *implemented as features of the application*.

### 5. Security & Governance (Implemented Features)
* **Authentication & Authorization (Implemented in Application):**
    * Use OAuth2/OIDC (users), RBAC/ABAC (permissions) *implemented by or integrated into the application*. Per `plan.md` JWT/MCP RBAC tasks. JWT: short TTL, refresh tokens, asymmetric signing *logic in the auth module*. MCP auth: explore `mcp.server.auth`, `OAuthServerProvider` *for implementation*.
* **Data Protection & Governance (Implemented in Application):**
    * Classify data (P0-P3); enforce AES-256 encryption, access controls (RBAC/ABAC/MFA) *within the application's data handling logic*. Encrypt data at rest/transit (TLS 1.2+) *configured or handled by the application*. Manage keys via KMS (regular rotation; Vault setup per `plan.md`) *through application integration with KMS/Vault*. Enforce retention *through application logic*. Maintain compliance maps (GDPR, etc.). GDPR/CCPA access/forget endpoints *implemented as API features*.
* **Dependency & Secret Management (Runtime Aspects):**
    * Never commit secrets; use vaults (Vault setup per `plan.md`), inject at runtime (the application needs to be able to read injected secrets). Config via env vars or files (application reads these at runtime).
* **Network Security (Application-Level Implementation):**
    * Restrict ingress/egress (if controlled by app config/logic); zero-trust (micro-segmentation, mTLS) *if the application implements these communication patterns*.
* **Code Security & Testing (Implemented Safeguards):**
    * **Critical for MCP tools**: Extreme care with FS/command exec; sanitize all inputs *within the tool's implementation* (guideline for `plan.md` tools).

### 6. AI/LLM Specific Standards (Implemented Features)
* **Prompt Hygiene (Implemented Logic):**
    * Structured prompts (JSON schema), strip secrets *as part of the application's LLM interaction logic*. 90% context window budget *enforced by the context assembly logic*.
* **Token Efficiency (Implemented Logic):**
    * Chunk scoring, context compression (per `plan.md` RAG components) *implemented as part of the RAG pipeline*.
* **Model Selection & Safety (Implemented Features):**
    * Production models (SLA/SOC-2); OSS models in isolated envs. Moderation endpoints for user prompts *called by the application*; log prompts/responses (30 days) *by the application's logging system*. LLM guardrails per `plan.md` *implemented in the interaction layer with LLMs*.
* **Fine-Tuning & RAG (Implemented System):**
    * Fine-tune on P1 or lower data (if approved). Deterministic embedding models *used by the system*. RAG implementation per `plan.md` (the actual RAG codebase).
* **AI/LLM Safeguards (Implemented Features):**
    * Fail fast on build/test/lint failures; escalate to humans. LLM PRs: same review as human PRs. Human-in-the-Loop (HITL) for critical automated decisions *if this logic is built into the application's workflow*.
* **Ethical AI Review & Bias Mitigation (Implemented Techniques):**
    * Implement techniques to mitigate RAG/LLM output bias *in the application's response generation or post-processing*.

## B. `codeRAG-mcp` Project-Specific Core Implementation (Part 2)

### 1. Project Overview & Core Mandates (Part 2.A - Implemented Requirements)
* **Project Scope:** `codeRAG-mcp` is a production-grade MCP server for context-aware code RAG in IDEs/Claude Desktop, using `modelcontextprotocol/python-sdk` (details in `plan.md`) (The server itself is the implementation).
* **Core Mandates for `codeRAG-mcp` (derived from `plan.md`) (These are features to be implemented):**
    * Ensure multi-session, non-eroding context integrity (per `plan.md` context tasks) *through implemented mechanisms*.
    * Meet advanced RAG requirements and deliver capabilities per `plan.md` *through implemented RAG logic*.
    * **Crucial: 100% Anthropic MCP & Claude Desktop compatibility (key `plan.md` deliverable) *achieved via correct MCP implementation***.
    * Implement robust audit logs, security (per `plan.md`), CI-driven QA. Meet Part 1.A test coverage (audit logs & security are implemented features).
    * Architecture: Support planned extensibility (`plan.md` P3) with maintainability & security (the implemented architecture should reflect this).
    * Services: Production-grade observability, failover, self-healing (per `plan.md` tasks) (these are implemented characteristics of the server).
    * Strictly no secrets/keys/sensitive data in codebase (complements `plan.md` security) (a property of the codebase).
* **Approved Tech Stack (Part 2.A - Technologies Used in Implementation):**
    * Core: Python 3.11+, `modelcontextprotocol/python-sdk` (`FastMCP`) *used to build the server*.
    * Pkg Mgmt: `uv`. Web: FastAPI (via `FastMCP`) *used for server implementation*. Validation: Pydantic *used for data models in code*.
    * Stores: Qdrant (primary vector), Chroma (fallback vector), Neo4j (graph) *clients and interaction logic implemented*.
    * Cache: Redis/Valkey *integration and caching logic implemented*. VCS: GitPython *used in code*. Parsing: Tree-sitter *used for chunking logic*. CLI: Typer/Click *for the application's CLI if any*.
    * Testing: Pytest, Hypothesis *used to write test code*. Lint/Format: Black, Ruff.
    * CI/CD/Container: Docker, GitHub Actions. Observability: Prometheus, Grafana, OpenTelemetry *integrated with/for the application*. Security Scan: Syft, Trivy, Semgrep.
* **Forbidden:** No secrets/keys/PII in code. No JS/TS in core RAG/MCP backend (Node.js/TS for sandboxed SDK testing/IDE plugin compat. only if vital). No untested/uncovered code below Part 1.A thresholds. No ambiguous deps, terms, workflow (These define qualities of the final codebase).

### 2. Context & Graph Principles (Part 2.B - Implemented Logic)
(Guiding principles for `plan.md` context/graph features that are implemented in code)
* **Context Persistence (NO DATA LOSS):** Read latest state; strictly continue from last context. Create explicit handoff file before exit. Validate persistence; halt if not 100% restorable. Define & test RPO/RTO for "Zero-Loss" (These are all behaviors and mechanisms implemented in the server).
* **Graph-Aware Hybrid Context (Implemented Features):**
    * **Incremental Graph Updates:** Changes reflect immediately; update only affected elements (The logic for this is implemented).
    * **Customizable Graph Schema:** Model code entities (nodes/edges) with metadata; design for extensibility (The schema and modeling are part of the data layer implementation).
    * **Hybrid Retrieval:** Fuse results (RRF); construct context via intelligent graph traversal (The fusion and traversal logic is implemented).
    * **Temporal Graph:** Store timestamps/hashes; support point-in-time reconstruction (The storage and reconstruction logic is implemented).
* **Advanced Context & Token Efficiency (Implemented Features):** Optimize context injection (relevance, proximity, summarization, AST-pruning, dedupe, graph-aware selection). **Zero-Loss Graph/Session Context:** Use immutable, versioned history (diffs, rollback where appropriate) (These optimization and versioning techniques are implemented in code).

### 3. MCP Implementation Guidance (Part 2.C - Core Server Implementation)
(Guidance for `plan.md` MCP server tasks, all pertaining to core implementation)
* **MCP Core Concepts (Implemented in Server):**
    * **Model**: Client-Server (build Server for Clients like LLMs, IDEs) (The server is the implementation).
    * **Transports**: `stdio` (local/IDE), HTTP/SSE (remote). `FastMCP` abstracts this (The server implements/uses these transports via FastMCP).
    * **Capabilities** (via `modelcontextprotocol/python-sdk`) (These are implemented server features):
        * **Resources (`@mcp.resource`)**: Read-only/low-side-effect data endpoints (like GET). Fast, ideally idempotent. Clear URIs (e.g., `files://{filepath}`). Graceful errors. (These are Python functions decorated and implemented in the server).
            ```python
            # Example: @mcp.resource("system://os-info") def get_os_info() -> dict: ...
            ```
        * **Tools (`@mcp.tool`)**: AI-invokable actions, potential side-effects. (These are Python functions decorated and implemented in the server).
            * **CRITICAL**: Excellent, detailed docstrings (purpose, args, behavior for LLM) *in the tool's code definition*.
            * Use Python type hints (for validation, LLM description) *in the tool's signature*.
            * Robust error handling, informative messages *returned by the tool's code*. Atomic, focused *design of the tool's implementation*.
            * **Idempotency is key for state-changing tools (per `plan.md`) *implemented in the tool's logic***.
            ```python
            # Example: @mcp.tool() def write_file(filepath: str, content: str) -> str: """Docstring...""" ...
            ```
        * **Prompts (`@mcp.prompt`)**: Reusable templates for interactions/system prompts (Implemented and registered with the server).
* **Server Implementation with `FastMCP` (Core Code):**
    * **Use `FastMCP`**: Recommended approach per `plan.md` (The server code uses this library).
        ```python
        # main.py (simplified)
        from mcp.server.fastmcp import FastMCP, Context
        import logging, platform, os, asyncio

        # Structured logging in production is vital
        logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        logger = logging.getLogger(__name__)

        mcp_server = FastMCP(title="CodeRAGMCPServer", version="0.1.0", description="CodeRAG MCP server.")

        @mcp_server.resource("system://os-info")
        async def get_os_info() -> dict: # Async for consistency
            logger.info("Executing get_os_info resource.")
            return {"system": platform.system(), "release": platform.release()}

        @mcp_server.tool()
        async def write_file_example(filepath: str, content: str, ctx: Context) -> str:
            """Writes content to filepath. Overwrites. Reports progress. Aims for idempotency."""
            logger.info(f"Tool write_file for {filepath}")
            try:
                await ctx.report_progress(0, 100, "Starting write...")
                def _bwrite(): # Blocking part
                    pdir = os.path.dirname(filepath)
                    if pdir: os.makedirs(pdir, exist_ok=True) # Create parent dir if needed
                    with open(filepath, "w", encoding='utf-8') as f: f.write(content)
                await asyncio.to_thread(_bwrite)
                await ctx.report_progress(100, 100, "Write complete.")
                return f"Successfully wrote to {filepath}"
            except Exception as e: # Catch specific IOErrors too
                logger.error(f"Error in write_file for {filepath}: {e}", exc_info=True)
                await ctx.report_progress(0, 100, f"Error: {e}")
                return f"Error writing to {filepath}: {str(e)}"
        
        # from . import your_tools; mcp_server.include_router(your_tools.router) # Example import

        if __name__ == "__main__": mcp_server.run()
        ```
    * **`Context` Object**: For progress, server config, calling other MCP capabilities. Add `ctx: Context` to tool/resource args *in their Python function signatures*.
        ```python
        # @mcp.tool() async def process_large_file(filepath: str, ctx: Context) -> str: ... await ctx.report_progress(...) ...
        ```
    * **Async Ops**: Use `async def` for all tools/resources *in their Python definitions*. For blocking/CPU-bound tasks, use `await asyncio.to_thread(...)` *in the implementation*.
    * **Config**: Env vars (`python-dotenv` local) or config files (Pydantic models per `plan.md`). No hardcoding (The application reads and uses this config at runtime).
    * **Auth**: If needed, use `mcp.server.auth`, `OAuthServerProvider` (per `plan.md`) *by integrating these into the server code*.
* **Strict Protocol Adherence (Implemented Behavior):** Follow Anthropic MCP Guidelines, examples (MCP compliance verification in `plan.md`). Implement tools per MCP contract. Robust versioning, validation, error handling *in the server code*. Plug-and-play for Claude Desktop/IDEs (achieved through correct implementation).
* **Modularity Principles (Implemented Architecture):**
    * Adapter-based graph storage (per `plan.md`) *implemented in the data access layer*.
    * Extensible analysis, chunking, retrieval via plugins; avoid hardwiring (The plugin system and modular design are implemented features).
    * Plugins: Safe sandboxing, versioned APIs, RBAC, audit (per `plan.md` plugin system) (These are features of the implemented plugin system).
    * Design components (RAG strategies, VDBs, parsers) for plug-and-play config. Use feature flags (per `plan.md`) for experiments (The code supports this configurability).
* **Future-Proofing (Implemented Design Considerations):** Version schemas/APIs; include migration tools. Support runtime modes (`full`, `standard`, `lite`). Integrate feedback/diagnostics hooks *into the application code*.

### 4. Testing & Observability Guidance (Part 2.D - Implemented Aspects)
* **Testing & Validation (`codeRAG-mcp` & MCP Specific - Test Code Artifacts):**
    * **Unit Tests**: `pytest` for isolated tool/resource tests. Mock dependencies (The actual `.py` test files).
    * **Debug Logging**: Comprehensive, structured *logging calls within the application code*; log inputs, outputs, errors, key decisions.
    * **Error Scenario Tests**: Explicitly test invalid inputs, failures; ensure graceful error reporting *by the application* (The test cases and application's response to them).
    * **Coverage**: Meet Part 1.A targets for the implemented code.
    * **E2E Tests (`plan.md`)**: Cover graph, hybrid, multi-repo, context handoff for `codeRAG-mcp` (The test scripts for these scenarios).
    * **Failure Simulation**: Test graceful degradation, informative errors *from the application*.
    * **Crash Dumps**: On critical failures, dump heap, graph state, context for post-mortem (The mechanism to create these dumps).
* **Observability (`codeRAG-mcp` & MCP Specific - Implemented Features):**
    * Log significant MCP events (crucial for debug/audit) *from the application code*.
    * Prometheus Metrics: Track MCP usage, times, errors, cache ratios, tokens, fallbacks (supports `plan.md` setup) *by implementing metric collection and exposure in the server*.
    * Provide API/CLI for context/graph lineage tracing *as an implemented feature of the server*.
    * Implement automated context/graph drift detection; alert/block on deviations *within the application or its monitoring integration*.

### 5. MCP Compliance & Best Practices (Part 2.E - Implemented in Code)
* **Idempotency:** Core `codeRAG-mcp` principle (per `plan.md`). Aim for idempotent resources/tools *by implementing them that way*.
* **Security:** Sanitize inputs (esp. for FS/command tools). Use proper auth for sensitive capabilities *as implemented in the server and tools*.
* **Follow SDK Examples:** Study official [MCP Python SDK examples](https://github.com/modelcontextprotocol/python-sdk/tree/main/examples) for best practices/advanced patterns *to apply in the implementation*.

---

## III. Glossary (Part 5)
* **MCP**: Model Context Protocol.
* **`FastMCP`**: SDK framework for MCP servers (FastAPI-based).
* **`@mcp.tool`/`@mcp.resource`/`@mcp.prompt`**: SDK decorators for MCP capabilities.
* **`Context` (MCP)**: Injected object for MCP server features (e.g., progress).
* **`mcp inspector`**: CLI for interactive MCP server testing.
* **stdio transport**: MCP communication via standard I/O.
* **RAG**: Retrieval-Augmented Generation.
* **Claude Desktop**: Anthropic app supporting MCP.
* **`uv`**: Fast Python package manager.
* **Idempotency**: Core `codeRAG-mcp` principle (per `plan.md`): multiple calls of an operation have same effect as one.
* **Phase Plan**: Phased approach in `plan.md`; PHASEs are atomic, testable deliverables.
* **RRF**: Reciprocal Rank Fusion (search result combination).
* **SLI/SLO**: Service Level Indicator/Objective.
* **SAST/DAST**: Static/Dynamic Application Security Testing.

---
