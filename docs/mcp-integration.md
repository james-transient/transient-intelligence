I guess # TI MCP Integration Guide

Use this guide to connect MCP-compatible clients to TI.

## Setup pattern

TI MCP runs as a local stdio process. Add a server entry and pass `TI_API_KEY` via environment variables.

Example shape:

```json
{
  "mcpServers": {
    "transient-intelligence": {
      "command": "node",
      "args": ["/path/to/ti-mcp-stdio.mjs"],
      "env": {
        "TI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

## Default tool workflow

1. `ti_workflow_policy` once at session start.
2. `ti_run_workflow` for primary upload-and-ask operations.
3. Follow retry instructions when indexing is still running.

## Manual tool workflow

- `ti_upload`
- `ti_run_status` (if retrieval is not ready)
- `ti_ask`

## Transport decision table

- Have extracted text/JSON: use `ti_upload` (or `ti_run_workflow` with text).
- Have HTTPS URL: use `ti_fetch_upload` (or `ti_run_workflow` with `input_url`).
- Have sandbox/local mount path only: use `ti_create_upload_handoff`.

## Response handling

- Always check `ok` before processing output.
- If `ok` is false, surface `error` and `next_steps`.
- If indexing is in progress, wait and retry based on returned guidance.
