# Project Context

- **Owner:** hahuber
- **Project:** Azure Search OpenAI Demo — RAG application for searching and chatting with your own data using Azure AI Search and Azure OpenAI
- **Stack:** Python (Quart), TypeScript (React/Vite), Bicep (Azure IaC), Azure Functions, Azure AI Search, Azure OpenAI
- **Created:** 2026-02-17

## Learnings

<!-- Append new learnings below. Each entry is something lasting about the project. -->

### 2026-02-17: Initial test suite review

**Test Architecture:**
- All tests in `tests/` directory, flat file structure (no subdirectories for test categories)
- Three test styles: unit tests (`test_*.py`), app integration tests (`test_app.py`, `test_cosmosdb.py`, `test_upload.py`, `test_content_file.py`), and e2e Playwright tests (`e2e.py`)
- pytest framework with `pytest-asyncio` for async, `pytest-snapshot` for golden-file comparisons, `pytest-playwright` for e2e
- pytest config in `pyproject.toml`: pythonpath includes `app/backend`, `scripts`, `app/functions`

**Test Infrastructure:**
- `conftest.py` (1157 lines) — massive shared fixture file; contains mock environment configs, client fixtures, search/blob/auth mocks
- `mocks.py` (687 lines) — mock classes for Azure services (credentials, blob, search, transport, speech, vision, agentic retrieval)
- Snapshot tests use `pytest-snapshot` with golden files in `tests/snapshots/`
- Parametrized fixtures create multiple test variants (e.g., `client` runs against both OpenAI and Azure OpenAI configs)

**Mocking Strategy:**
- Mocks at class/method level via `monkeypatch.setattr` — consistent pattern throughout
- `MockAzureCredential`, `MockTransport`, `MockAsyncSearchResultsIterator` are core shared mocks
- Environment variables controlled via `mock.patch.dict(os.environ, clear=True)` + `monkeypatch.setenv`
- `AzureDeveloperCliCredential` is patched to return `MockAzureCredential` in all client fixtures
- OpenAI mocking uses closure-based patchers (`mock_openai_chatcompletion` returns a `patch` function)

**Key file paths:**
- `tests/conftest.py` — all shared fixtures, env configs, client factories
- `tests/mocks.py` — mock classes (credentials, blob, search, transport, speech, retrieval)
- `tests/e2e.py` — Playwright e2e tests (11 tests), runs real server via multiprocessing + uvicorn
- `tests/test_app.py` — primary app integration tests (58+ tests across ask/chat/stream/vision/auth)
- `tests/test_chatapproach.py` — unit tests for approach logic (search query extraction, followups, ref_id replacement, embeddings)
- `tests/test_agentic_retrieval.py` — unit tests for agentic retrieval (sorting, web/sharepoint results, minimal reasoning)
- `tests/test_function_apps.py` — tests for Azure Functions (document_extractor, figure_processor, text_processor)
- `tests/test_searchmanager.py` — tests for search index creation/updates
- `tests/test_blob_manager.py` — tests for blob upload/download (some skipped on Windows)
- `tests/test_cosmosdb.py` — chat history CRUD tests
- `tests/test_upload.py` — user file upload tests
- `tests/test_authenticationhelper.py` — auth claims, token validation, path auth
- `tests/test_prepdocs.py` — embedding computation, rate limiting tests
- `tests/test_app_config.py` — config endpoint and app startup config tests
- `tests/test_mediadescriber.py` — Content Understanding and multimodal describer tests

**Coverage gaps identified:**
- No dedicated test file for `retrievethenread.py` (tested indirectly via test_app.py `/ask` endpoints)
- No tests for `promptmanager.py` (PromptyManager — template loading/rendering)
- No tests for `decorators.py` (`@authenticated`, `@authenticated_path`)
- No tests for `error.py` (`error_dict`, `error_response` — partially tested via test_app exception tests)
- No tests for `config.py` (just constants, acceptable)
- No tests for `core/sessionhelper.py` (`create_session_id`)
- No tests for `custom_uvicorn_worker.py` or `gunicorn.conf.py`
- No tests for `main.py` entry point
- `cloudingestionstrategy.py` in prepdocslib has no test file
- `integratedvectorizerstrategy.py` in prepdocslib has no test file
- `figureprocessor.py` partially tested via test_function_apps.py but no dedicated unit tests

**Flakiness risks:**
- E2e tests use real server process + port binding — potential for port conflicts or timing issues
- `wait_for_server_ready` has 10s timeout with 0.5s intervals — could fail on slow CI
- Blob manager tests skip on Windows due to `NamedTemporaryFile` handle issues
- E2e tests use `try/except` in route handlers and `print()` for errors — assertion failures in handlers are *swallowed* and won't fail the test

📌 Team update (2026-02-17): Approach base class and setup_clients() refactoring planned — test updates will be needed for changed backend module structure — decided by Keaton
