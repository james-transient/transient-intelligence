# Solving AI Hallucination: TI

Transient Intelligence (TI) is an evidence-first document analysis platform for developers. It reduces hallucination risk by ranking evidence and returning citation-grounded outputs through stable API and MCP integration contracts.

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

1. Create an account and generate an API key.
2. Call the one-call orchestrator endpoint.
3. Enforce citation checks in your integration logic.

```bash
curl -X POST "https://api.transientintelligence.com/api/models/v1/answer" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question":"What was Q3 revenue?","input":"Q3 revenue reached $1.2M.","top_k":5}'
```

Minimal integration rule:

- If `citations` is empty, return "No evidence found" and do not synthesize unsupported claims.

## Core API Surfaces

- `POST /api/models/v1/answer` - one-call upload + retrieval + answer (recommended default)
- `POST /api/models/v1/upload` - ingest source data and return `session_id` (and optional `run_id`)
- `POST /api/models/v1/ask` - query a retrieval-ready session
- `GET /api/models/v1/runs/{id}` - poll async run status
- `GET /api/models/v1/runs/{id}/results` - fetch final run output

## MCP Integration (TI Tools)

Preferred path:

1. Call `ti_workflow_policy` once at session start.
2. Use `ti_run_workflow` as default tool.
3. Follow `action_required` / retry signals when indexing is in progress.

Manual path:

- `ti_upload` -> `ti_run_status` -> `ti_ask`

## Guides

- [Quickstart guide](docs/quickstart.md)
- [API integration guide](docs/api-integration.md)
- [MCP integration guide](docs/mcp-integration.md)

## Public-Safe Scope

This repository is intentionally limited to public-safe, developer-facing material. Proprietary model internals, private infrastructure details, and undisclosed algorithms are not included.

## Related Project

- Transient Recall (TR): [transient-recall](https://github.com/james-transient/transient-recall)
