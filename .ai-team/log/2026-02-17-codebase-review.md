# Session: Team Onboarding and Initial Codebase Review

- **Date:** 2026-02-17
- **Requested by:** hahuber
- **Participants:** Keaton (Lead), Hockney (Frontend Dev), Fenster (Backend Dev), McManus (Tester)

## Summary

Full codebase review conducted as part of team onboarding. Each agent reviewed their domain area and filed decisions to the inbox.

## Agent Activity

- **Keaton (Lead):** Reviewed overall architecture. Identified strategy-based RAG pattern strengths, approach base class bloat (1016 lines), setup_clients monolith (~340 lines), duplicated code between approach subclasses, constructor parameter explosion, and untyped config keys.
- **Hockney (Frontend Dev):** Reviewed frontend code. Identified Chat.tsx god-component (~669 lines, ~40 useState hooks), hardcoded "Retry" string in AnswerError.tsx, `any` type usage in API models, missing React Error Boundary, and stringly-typed Settings onChange handler.
- **Fenster (Backend Dev):** Reviewed backend and infrastructure. Confirmed setup_clients monolith and approach duplication findings. Additionally identified error handling gaps in /delete_uploaded and /speech routes, manual prepdocslib copy sync risk, missing /health endpoint, and Bicep main.bicep size (1507 lines).
- **McManus (Tester):** Reviewed test suite and coverage. Identified e2e test assertion swallowing (try/except silences failures), conftest.py at 1157 lines needing decomposition, missing unit tests for decorators.py/promptmanager.py/sessionhelper.py, and blob manager tests skipped on Windows.

## Key Findings

1. approach.py base class (1016 lines) accumulates too many responsibilities
2. setup_clients() is a ~340-line monolith needing decomposition
3. Chat.tsx is a god-component with ~40 useState hooks
4. E2E test assertions are silently swallowed by try/except in route handlers
5. Test coverage gaps: decorators.py, promptmanager.py, sessionhelper.py have zero dedicated tests

## Decisions Filed

4 decision files merged from inbox into decisions.md.
