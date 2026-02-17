# Project Context

- **Owner:** hahuber
- **Project:** Azure Search OpenAI Demo — RAG application for searching and chatting with your own data using Azure AI Search and Azure OpenAI
- **Stack:** Python (Quart), TypeScript (React/Vite), Bicep (Azure IaC), Azure Functions, Azure AI Search, Azure OpenAI
- **Created:** 2026-02-17

## Learnings

<!-- Append new learnings below. Each entry is something lasting about the project. -->

### 2026-02-17: Full Architecture Review

**Stack:** Python 3.10+ (Quart async framework), React/TypeScript (Vite), Bicep IaC, Azure Functions for cloud ingestion.

**Architecture Pattern — Strategy-based RAG:**
- `Approach` base class (1016 lines, [approach.py](app/backend/approaches/approach.py)) contains ALL shared retrieval logic: search, agentic retrieval, embedding computation, blob download, query rewriting, chat completion creation.
- Two concrete strategies: `RetrieveThenReadApproach` (ask, single-turn) and `ChatReadRetrieveReadApproach` (chat, multi-turn with query rewriting).
- Both strategies share `run_agentic_retrieval_approach` and `run_search_approach` from the base class. The chat approach adds streaming support and follow-up question extraction.
- Prompts managed via `PromptyManager` using `.prompty` files — clean separation of prompt templates from code.

**Key File Paths:**
- `app/backend/app.py` (839 lines): Quart app, all routes, massive `setup_clients()` function (~350 lines) that wires everything together.
- `app/backend/config.py` (43 lines): String constants for `current_app.config[]` keys — acts as a typed registry without actual typing.
- `app/backend/decorators.py`: `@authenticated` and `@authenticated_path` decorators for route auth.
- `app/backend/error.py`: Central error formatting with OpenAI-specific error classification (content filter, context length).
- `app/backend/main.py`: Entry point, loads azd env vars when not on Azure.
- `app/backend/prepdocslib/`: Document ingestion library — parsers, chunkers, embeddings, blob managers, search managers.
- `app/frontend/src/api/models.ts`: TypeScript types mirroring backend request/response shapes.
- `infra/main.bicep` (1507 lines): Complete Azure provisioning — AI Search, OpenAI, Storage, App Service/Container Apps, CosmosDB, Speech, Document Intelligence, monitoring.
- `tests/conftest.py` (1157 lines): Extensive mock fixtures for Azure services.

**Architecture Decisions Observed:**
1. **Quart (async Flask)** chosen for Python async support with Azure SDK — correct choice.
2. **Config via `current_app.config` dict** with string constants — works but not type-safe. No dependency injection container.
3. **Constructor injection** for approach classes — good, but constructors now take 20+ parameters each due to feature growth (knowledgebase clients ×4, blob managers ×2, etc.).
4. **Single `setup_clients()` function** handles ALL initialization — becoming a maintenance bottleneck at ~350 lines.
5. **Duplicated `_select_knowledgebase_client` and `run_agentic_retrieval_approach`** in both `RetrieveThenReadApproach` and `ChatReadRetrieveReadApproach` — these are copy-pasted, not inherited from the base class.
6. **Both approach subclasses bypass the base `__init__`** — they each set all instance vars manually instead of calling `super().__init__()`. This means the base class `__init__` parameters drift from what the subclasses actually use.
7. **prepdocslib is duplicated** into Azure Functions directories via `scripts/copy_prepdocslib.py` — pragmatic for deployment but creates sync risk.
8. **Comprehensive test infrastructure** — mocks at HTTP level, good coverage strategy documented in AGENTS.md.
9. **Monitoring via OpenTelemetry + Azure Monitor** — properly instrumented (aiohttp, httpx, OpenAI SDK, ASGI middleware).
10. **CORS, authentication, access control** all configurable via environment variables — defensive defaults (disabled until explicitly enabled).
11. **Frontend config route (`/config`)** dynamically tells the UI which features are available — good pattern for feature flags.
12. **Infrastructure uses Bicep modules** in `infra/core/` and `infra/app/` — well-organized, parameterized.
13. **GPT reasoning model support** tracked in a static dict with streaming/minimal_effort flags — will need ongoing maintenance as models evolve.

📌 Team update (2026-02-17): Chat.tsx identified as god-component (~669 lines, ~40 useState) needing state extraction into custom hooks — decided by Hockney
📌 Team update (2026-02-17): Fenster confirms setup_clients() monolith and approach duplication findings; also identified error handling gaps and missing /health endpoint — decided by Fenster
