# Solving AI Hallucination: TI

Transient Intelligence (TI) is an evidence-first document analysis platform for developers. It reduces hallucination risk by ranking evidence and returning citation-grounded outputs through stable API and MCP integration contracts.

## Legal Notice

- All rights reserved.
- Public docs/examples only.
- No rights granted to TI proprietary implementation.

## Website

- [Create account](https://transientintelligence.com/auth/register)
- [Log in](https://transientintelligence.com/auth/login)
- [Dashboard & API keys](https://transientintelligence.com/dashboard/developers)
- [Docs](https://transientintelligence.com/docs)

## Why TI

- Evidence-first responses with explicit citations
- Deterministic integration paths for API and MCP clients
- Production-ready endpoint contracts at `https://api.transientintelligence.com`

## Developer Quickstart

1. **Get API key** — Create an account, log in, generate a key from the [Developers dashboard](https://transientintelligence.com/dashboard/developers).
2. **First request** — Call the one-call orchestrator endpoint.
3. **Enforce citations** — If `citations` is empty, return "No evidence found"; do not synthesize unsupported claims.
4. **Manage sessions carefully** — Treat `session_id` as review-scope state. Reuse only for the same evidence set; start a new session for new documents or review goals.

```bash
curl -X POST "https://api.transientintelligence.com/api/models/v1/answer" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question":"What was Q3 revenue?","input":"Q3 revenue reached $1.2M.","top_k":5}'
```

## Core API surfaces

| Endpoint | Purpose |
|----------|---------|
| `POST /api/models/v1/answer` | One-call upload + retrieval + answer (recommended default) |
| `POST /api/models/v1/upload` | Ingest source data, return `session_id` |
| `POST /api/models/v1/ask` | Query a retrieval-ready session |
| `GET /api/models/v1/runs/{id}` | Poll async run status |
| `GET /api/models/v1/runs/{id}/results` | Fetch final run output |

## Supported input types

`.pdf`, `.docx`, `.xlsx`, `.csv`, `.txt`, `.json`

## MCP integration (TI tools)

Use MCP when you want AI clients (Cursor, Claude Desktop, ChatGPT Desktop, or other MCP-compatible tools) to run evidence-grounded Q&A workflows without custom API wiring.

**Preferred workflow-run path:**

1. Call `ti_workflow_policy` once at session start.
2. Use `ti_run_workflow` as default tool.
3. Follow `action_required` / retry signals when indexing is in progress.

**Flow:** `ti_workflow_policy` -> `ti_run_workflow` -> citations returned

**`ti_run_workflow` examples:**

- Inline text: ask + ingest in one call
- HTTPS file URL: upload and query a document in one call
- Existing session: query previously indexed content by `session_id`

**Manual path:** `ti_upload` → `ti_run_status` → `ti_ask`

See full MCP setup, client config, and transport decision table in [MCP integration guide](docs/mcp-integration.md).

## Guides

- [Quickstart guide](docs/quickstart.md) — API key, first request, cURL/JS/Python, AI integration prompt
- [API integration guide](docs/api-integration.md) — Endpoints, request params, response handling
- [MCP integration guide](docs/mcp-integration.md) — Tool workflow, transport resolver
- [Claude Desktop cloud setup](docs/claude-desktop-cloud-setup.md) — User onboarding for hosted TI MCP

## Public-Safe Scope

This repository is intentionally limited to public-safe, developer-facing material. Proprietary model internals, private infrastructure details, and undisclosed algorithms are not included.

## Related Project

- Transient Recall (TR): [transient-recall](https://github.com/james-transient/transient-recall)
