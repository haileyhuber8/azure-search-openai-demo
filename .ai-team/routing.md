# Work Routing

How to decide who handles what.

## Routing Table

| Work Type | Route To | Examples |
|-----------|----------|----------|
| Architecture, scope, priorities | Keaton | What to build next, trade-offs, cross-cutting decisions, code review |
| Frontend UI, React, TypeScript | Hockney | Components, pages, API client, i18n, Vite config |
| Backend API, Python, Quart | Fenster | Approaches, prompts, prepdocslib, app.py, Azure integrations |
| Infrastructure, Bicep, Azure | Fenster | Bicep templates, azd config, deployment |
| Testing, quality, coverage | McManus | Write tests, find edge cases, verify fixes, e2e tests |
| Code review | Keaton | Review PRs, check quality, suggest improvements |
| Session logging | Scribe | Automatic — never needs routing |

## Issue Routing

| Label | Action | Who |
|-------|--------|-----|
| `squad` | Triage: analyze issue, assign `squad:{member}` label | Keaton |
| `squad:keaton` | Architecture/review work | Keaton |
| `squad:hockney` | Frontend work | Hockney |
| `squad:fenster` | Backend/infra work | Fenster |
| `squad:mcmanus` | Testing work | McManus |
