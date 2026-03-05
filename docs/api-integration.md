# Transient Intelligence — API Integration

Everything you need to integrate the Transient Intelligence API into your product — endpoints, authentication, request shapes, and response handling. Full docs: [transientintelligence.com/docs/api-integration](https://transientintelligence.com/docs/api-integration)

## Tools & Artifacts

Download official API specifications and test collections from the docs site:

- **Postman Collection** — [TI01_V1_Postman_Collection.json](https://transientintelligence.com/TI01_V1_Postman_Collection.json) — Pre-configured requests and auth
- **OpenAPI Spec** — [TI01_V1_OPENAPI.yaml](https://transientintelligence.com/TI01_V1_OPENAPI.yaml) — YAML schema for auto-generation

## Authentication

All API endpoints require the `x-api-key` header. Every request must include it.

```
x-api-key: <YOUR_TI_API_KEY>
```

- [Create account](https://transientintelligence.com/auth/register) · [Log in](https://transientintelligence.com/auth/login) · [Get API key](https://transientintelligence.com/dashboard/developers)

## Core Endpoints

Two integration paths:

- **`/answer`** — for inline text; handles upload, indexing, and query in one call (recommended default)
- **`/upload` → `/ask`** — decoupled flow for large files or async workloads

### POST /api/models/v1/answer (Recommended default)

One-call orchestrator. Accepts inline content and a question, handles upload and evidence retrieval internally, returns a grounded response. Equivalent to `ti_run_workflow` in MCP.

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| question | string | Yes | The question to answer from the provided evidence |
| input | string | Conditional | Raw text or JSON content to ingest inline. Required unless session_id or input_url |
| input_url | string | Conditional | HTTPS URL to a publicly accessible file. Must not be a local path |
| session_id | string | No | Reuse an existing indexed session |
| top_k | integer | No | Evidence chunks to retrieve. Default: 10. Max: 50 |
| wait_max_ms | integer | No | Max ms to wait for async indexing. Default: 10000. Max: 60000 |
| retry_after_seconds | integer | No | Hint returned in retry payloads. Default: 10 |
| include_normative | boolean | No | Include normative_check and factual_confidence. Default: true |

**top_k guide:** 1–5 precision, 6–15 balanced, 16–50 recall.

### POST /api/models/v1/upload

Ingest source data into a session. Accepts JSON payloads and multipart file uploads. Returns `session_id` and optionally `run_id` for async tracking.

**Supported input types:** `.pdf`, `.docx`, `.xlsx`, `.csv`, `.txt`, `.json`

### POST /api/models/v1/ask

Evidence-first query endpoint. Requires `session_id` from a prior upload and a `question`. Returns grounded citations and optional synthesized answer. Only call once `retrieval_ready: true`.

### GET /api/models/v1/runs/{id}

Poll the status of an asynchronous run. Call after `/upload` returns `retrieval_ready: false`. Proceed to `/ask` once `status` is `completed`.

### GET /api/models/v1/runs/{id}/results

Retrieve the final output of a completed run.

## Response handling

**Completed:**

```json
{
  "success": true,
  "mode": "answer_completed",
  "sessionId": "ti-01-session-abc123",
  "query": "What was Q3 revenue?",
  "answer": { "summary": "Q3 revenue reached $1.2M." },
  "totalMatches": 3,
  "citations": [{ "doc_id": "inline_input", "snippet": "Q3 revenue reached $1.2M", "locator": "paragraph 1" }],
  "factual_confidence": { "label": "Strong", "message": "Answer fully supported by evidence." }
}
```

**Indexing in progress (retry required):**

```json
{
  "success": true,
  "mode": "answer_wait_then_retry",
  "action_required": "wait_then_retry",
  "session_id": "ti-01-session-abc123",
  "retry_after_seconds": 10,
  "next_action": "Indexing is still in progress. Retry /v1/answer with the same session_id shortly."
}
```

## Reliability guardrails

- Check `citations` before using answer content
- Handle `action_required: wait_then_retry` explicitly
- Tune `top_k`: 1–5 precision, 6–15 balanced, 16–50 recall
- Never hardcode API keys; use environment variables
