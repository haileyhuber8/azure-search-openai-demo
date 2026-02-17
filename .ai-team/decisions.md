# Team Decisions

<!-- Canonical decision log. Scribe merges entries from .ai-team/decisions/inbox/ -->

---

### 2026-02-17: Architecture Review — Findings and Recommendations
**By:** Keaton
**What:** Completed full architectural review covering structure, patterns, cross-cutting concerns, Azure integration, and code quality.
**Why:** Baseline assessment to identify strengths, risks, and improvement opportunities.

**Issues:**
1. `setup_clients()` is a ~350-line monolith — should be decomposed into focused setup functions.
2. Duplicated code between approach subclasses — `_select_knowledgebase_client()` and `run_agentic_retrieval_approach()` are copy-pasted instead of inherited.
3. Subclasses bypass `super().__init__()` — base class `__init__` exists but isn't called.
4. Constructor parameter explosion — 20+ params, 4 knowledgebase_client variants (combinatorial smell).
5. Config keys are untyped strings — no runtime validation for required keys.
6. `approach.py` base class is 1016 lines — too many responsibilities.

**Recommendations (priority order):**
1. Extract duplicated approach logic into base class or mixin — low risk, high value.
2. Break `setup_clients()` into focused setup functions — medium effort, high readability.
3. Have subclass `__init__` call `super().__init__()` — requires careful testing.
4. Consider typed config dataclass — medium effort, prevents runtime surprises.

---

### 2026-02-17: Chat.tsx needs state extraction
**By:** Hockney
**What:** Chat.tsx (~669 lines, ~40 useState hooks) should extract state into a custom hook (`useChatState`). Shared config/settings logic should be unified with Ask.tsx.
**Why:** Both pages duplicate config loading, settings change handlers, and API request construction.

---

### 2026-02-17: "Retry" button in AnswerError.tsx must be localized
**By:** Hockney
**What:** Hardcoded `"Retry"` string should use `t("retry")` with the key added to all 10 locale files.
**Why:** Only hardcoded English string in the frontend. Breaks localization contract.

---

### 2026-02-17: Reduce `any` usage in API models
**By:** Hockney
**What:** Replace `session_state: any`, `Thoughts.description: any`, `HistoryApiResponse.answers: any`, and `postChatHistoryApi(item: any)` with proper types or `unknown`.
**Why:** `strict: true` is enabled but these `any` escape hatches undermine it.

---

### 2026-02-17: Add a React Error Boundary at the app root
**By:** Hockney
**What:** No error boundary exists. A component crash white-screens the entire app.
**Why:** Table stakes for production React apps. Minimal effort, high payoff.

---

### 2026-02-17: Settings onChange handler should use a discriminated union
**By:** Hockney
**What:** Replace `onChange: (field: string, value: any) => void` with a discriminated union type.
**Why:** Zero compile-time safety with current string-based approach. Typos silently do nothing.

---

### 2026-02-17: Backend & Infrastructure Review — Findings and Recommendations
**By:** Fenster
**What:** Comprehensive backend and infrastructure review.
**Why:** Baseline assessment for backend domain.

**Key findings:**
1. `setup_clients()` monolith confirmed (~340 lines, 50+ env vars, 10+ clients).
2. Code duplication between Ask and Chat approaches confirmed.
3. `approach.py` base class too large (1016 lines) confirmed.
4. Error handling gaps: `/delete_uploaded` has no try/except; `/speech` returns generic error instead of `error_response()`.
5. prepdocslib copy sync to Azure Functions is manual — risk of stale copies.
6. No `/health` or `/readiness` endpoint.
7. Bicep infrastructure well-structured but `main.bicep` (1507 lines) could use more module extraction.
8. Streaming error handling, config management, and authentication are solid.

---

### 2026-02-17: E2E route handler assertions must not be swallowed
**By:** McManus
**What:** E2E tests wrap route handler assertions in try/except that silently swallows failures via `print()`. Tests can never fail on wrong request payloads.
**Why:** A test that can't fail isn't a test.

---

### 2026-02-17: conftest.py needs decomposition
**By:** McManus
**What:** `conftest.py` is 1157 lines. Should be split into logical modules or use a factory pattern for the 7+ near-identical `mock_*_env` fixtures.
**Why:** Maintenance hazard. Repeated patterns indicate a factory would be cleaner.

---

### 2026-02-17: Missing unit tests for decorators.py, promptmanager.py, sessionhelper.py
**By:** McManus
**What:** Three backend modules have zero dedicated test coverage.
**Why:** `decorators.py` contains auth enforcement. `promptmanager.py` drives LLM prompts. `sessionhelper.py` has branching logic.

---

### 2026-02-17: Blob manager tests are skipped on Windows
**By:** McManus
**What:** 4 tests skipped via `@pytest.mark.skipif(WINDOWS)` due to `NamedTemporaryFile` handle issues.
**Why:** Windows developers get no feedback on blob manager regressions.
