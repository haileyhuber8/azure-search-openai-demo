# Project Context

- **Owner:** hahuber
- **Project:** Azure Search OpenAI Demo — RAG application for searching and chatting with your own data using Azure AI Search and Azure OpenAI
- **Stack:** Python (Quart), TypeScript (React/Vite), Bicep (Azure IaC), Azure Functions, Azure AI Search, Azure OpenAI
- **Created:** 2026-02-17

## Learnings

<!-- Append new learnings below. Each entry is something lasting about the project. -->

### 2026-02-17: Initial Frontend Codebase Review

**Architecture & Key Paths:**
- Entry point: `app/frontend/src/index.tsx` — hash router, MSAL auth wrapper, i18n provider
- Layout shell: `src/layoutWrapper.tsx` → `src/pages/layout/Layout.tsx` (header, nav, outlet)
- Two main pages: `src/pages/chat/Chat.tsx` (streaming chat) and `src/pages/ask/Ask.tsx` (single Q&A)
- API layer: `src/api/models.ts` (types), `src/api/api.ts` (fetch wrappers), `src/api/index.ts` (barrel)
- Auth: `src/authConfig.ts` — top-level await for `/auth_setup`, exports `useLogin`, `msalConfig`, etc.
- i18n: `src/i18n/config.ts` — 10 languages, all statically imported, using i18next + browser detector
- Components: 19 component folders under `src/components/`. CSS Modules used throughout.
- History: Provider pattern (`IProvider.ts`) with IndexedDB, CosmosDB, and None implementations
- Settings: Massive shared `Settings.tsx` component used by both Chat & Ask pages

**Conventions observed:**
- Functional components only, no class components
- FluentUI v8 (`@fluentui/react`) is the primary UI library, with some v9 (`@fluentui/react-components`) usage in `QuestionInput`
- CSS Modules (`.module.css`) for scoped styling
- `const enum` for `RetrievalMode` and `HistoryProviderOptions`
- Lazy loading for Ask and NoPage routes
- `react-helmet-async` for per-page titles
- All user-facing strings use `useTranslation()` / `t()` — except one hardcoded "Retry" in AnswerError.tsx

**TypeScript:**
- `strict: true` in tsconfig — good
- 70 total `.ts`/`.tsx` source files, zero TypeScript errors
- Notable `any` usage: ~15 instances across models.ts, api.ts, Chat.tsx, Ask.tsx, Settings.tsx, AgentPlan.tsx, ThoughtProcess.tsx

**i18n:**
- 10 locales, 128 leaf keys in English baseline
- All locales match EN key set. A few locales have harmless extra keys (`helpTexts.llmInputs`, `labels.maxSubqueryCount`)
- One missing localization: "Retry" button text in `AnswerError.tsx` is hardcoded English

**Build:**
- Vite 6, React 18, TypeScript 5.6
- Output to `../backend/static` — backend serves the built SPA
- Manual chunk splitting for FluentUI icons, FluentUI react, and vendor
- Dev proxy forwards 12 API routes to backend on port 50505

**Concerns identified:**
1. Chat.tsx is a 669-line mega-component with ~40 useState hooks — candidate for state extraction
2. Chat.tsx and Ask.tsx share significant duplicated code (config loading, settings handling, API request building)
3. `handleSettingsChange` uses stringly-typed field dispatch — fragile, no type safety on field names
4. Mixed FluentUI versions (v8 + v9) — potential bundle bloat and visual inconsistency
5. `AnswerError.tsx` has hardcoded "Retry" string, not localized
6. `any` types in models.ts (`session_state`, `Thoughts.description`, `HistoryApiResponse.answers`) and api.ts (`postChatHistoryApi`) reduce type safety
7. No error boundary — a rendering crash in any component will white-screen the entire app
8. `authConfig.ts` uses top-level `await` — if `/auth_setup` fails, the entire app fails to load silently

📌 Team update (2026-02-17): E2E test route handler assertions are being swallowed by try/except — frontend e2e tests may pass despite wrong payloads — decided by McManus
