# Project Context

- **Owner:** hahuber
- **Project:** Azure Search OpenAI Demo — RAG application for searching and chatting with your own data using Azure AI Search and Azure OpenAI
- **Stack:** Python (Quart), TypeScript (React/Vite), Bicep (Azure IaC), Azure Functions, Azure AI Search, Azure OpenAI
- **Created:** 2026-02-17

## Learnings

<!-- Append new learnings below. Each entry is something lasting about the project. -->

### 2026-02-17: Full Backend & Infrastructure Review

**Backend Architecture:**
- Quart async framework served via Gunicorn + custom Uvicorn worker. Entry: `main.py` → `app.py:create_app()`.
- Single Blueprint with routes for `/ask`, `/chat`, `/chat/stream`, `/config`, `/auth_setup`, `/speech`, `/upload`, `/delete_uploaded`, `/list_uploaded`, `/content/<path>`.
- Chat history is a separate Blueprint (`chat_history_cosmosdb_bp`) in `chat_history/cosmosdb.py`.
- `@bp.before_app_serving` hook (`setup_clients()`) initializes ALL Azure clients, approaches, and config. ~340 lines in one function.
- Config keys are string constants in `config.py` (44 keys), stored in `current_app.config`.
- Authentication via `decorators.py` (`@authenticated`, `@authenticated_path`) wrapping `AuthenticationHelper` in `core/authentication.py`.
- Error handling in `error.py` — handles OpenAI `content_filter` and `context_length_exceeded` specifically, generic fallback otherwise. Streaming errors caught in `format_as_ndjson()`.
- CORS configured via `quart_cors` with `ALLOWED_ORIGIN` env var.
- OpenTelemetry + Azure Monitor integration when `APPLICATIONINSIGHTS_CONNECTION_STRING` is set.

**RAG Approaches:**
- Base class: `approach.py` (1016 lines) — contains `Document`, `WebResult`, `SharePointResult`, `DataPoints`, `ExtraInfo`, `ThoughtStep` dataclasses plus core `search()`, `rewrite_query()`, `run_agentic_retrieval()`, `create_chat_completion()`, `get_sources_content()` methods.
- `RetrieveThenReadApproach` (Ask, 318 lines): Single-turn Q&A. Searches, formats sources, sends to LLM with `ask_answer_question.prompty`.
- `ChatReadRetrieveReadApproach` (Chat, 526 lines): Multi-turn. First rewrites query via `chat_query_rewrite.prompty` + tool calling, then searches, then generates answer with `chat_answer_question.prompty`. Supports streaming.
- Both approaches support either standard Azure AI Search or Agentic Retrieval via `KnowledgeBaseRetrievalClient`.
- Agentic retrieval supports web + SharePoint knowledge sources with combination client selection logic.
- Prompts use `.prompty` format rendered via `prompty` library through `PromptyManager`.
- GPT reasoning model support tracked in `GPT_REASONING_MODELS` dict with streaming/minimal-effort flags.

**Document Ingestion (prepdocslib):**
- `FileStrategy`: Lists files (local or ADLS) → uploads to blob → parses → chunks → indexes into Azure AI Search.
- `UploadUserFileStrategy`: For user uploads via the app — writes to ADLS Gen2, indexes with ACLs.
- `CloudIngestionStrategy`: Wires Azure Function custom skills into an AI Search indexer (document extraction → figure processing → text processing).
- Parsers: `LocalPdfParser` (pypdf), `DocumentAnalysisParser` (Azure Document Intelligence), `LocalHTMLParser`, `CsvParser`, `JsonParser`, `TextParser`.
- Splitters: `SentenceTextSplitter`, `SimpleTextSplitter` — tiktoken-based, with CJK support.
- `FigureProcessor`: Extracts figures from PDFs, generates descriptions via OpenAI or Content Understanding.
- `SearchManager`: Creates/updates Azure AI Search indexes with vector search, semantic search, knowledge bases.
- `servicesetup.py`: Shared setup helpers used by BOTH the backend app and Azure Functions.
- `BlobManager` / `AdlsBlobManager`: Separate managers for global content and user-uploaded content.

**Azure Functions (Cloud Ingestion):**
- Three functions: `document_extractor`, `figure_processor`, `text_processor`.
- Each is an Azure Function custom Web API skill invoked by Azure AI Search indexer.
- They bundle a copy of `prepdocslib` (synced via `scripts/copy_prepdocslib.py`).
- Pattern: global settings object initialized on first request, uses `ManagedIdentityCredential`.

**Infrastructure (Bicep):**
- `infra/main.bicep`: 1507 lines, subscription-scoped. Provisions: resource groups, App Service/Container Apps, Azure OpenAI, AI Search, Storage, Document Intelligence, Content Understanding, Vision, Speech, Cosmos DB, Application Insights.
- Supports dual deployment targets: App Service and Container Apps.
- Uses Azure Verified Modules (AVM) from Bicep public registry for cognitive services and Cosmos DB.
- Extensive RBAC role assignments for both user principals and system identities.
- Private endpoint support with VPN gateway option.
- `main.parameters.json`: Maps azd env vars to Bicep params (364 lines).
- Conditional deployment: many modules gated on feature flags (useMultimodal, useUserUpload, useCloudIngestion, etc.).

**Key File Paths:**
- `app/backend/app.py` — 839 lines, main Quart app with routes and setup_clients
- `app/backend/approaches/approach.py` — 1016 lines, base approach with search/retrieval/LLM logic
- `app/backend/approaches/chatreadretrieveread.py` — 526 lines, multi-turn chat approach
- `app/backend/approaches/retrievethenread.py` — 318 lines, single-turn ask approach
- `app/backend/approaches/promptmanager.py` — PromptyManager for .prompty files
- `app/backend/prepdocslib/` — Document ingestion library (22 modules)
- `app/backend/prepdocslib/servicesetup.py` — 335 lines, shared service initializers
- `app/backend/prepdocslib/searchmanager.py` — 680 lines, Azure AI Search index management
- `app/backend/prepdocslib/textsplitter.py` — 609 lines, text chunking with tiktoken
- `app/backend/config.py` — Config key constants
- `app/backend/error.py` — Error response formatting
- `app/backend/decorators.py` — Auth decorators
- `app/backend/main.py` — Entry point, loads azd env
- `app/backend/gunicorn.conf.py` — Gunicorn config with custom Uvicorn worker
- `infra/main.bicep` — 1507 lines, main infrastructure template
- `infra/main.parameters.json` — 364 lines, parameter mappings

📌 Team update (2026-02-17): Approach base class refactoring and setup_clients() decomposition prioritized as top architectural improvements — decided by Keaton
📌 Team update (2026-02-17): Missing unit tests identified for decorators.py, promptmanager.py, sessionhelper.py — decided by McManus
