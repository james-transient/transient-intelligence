# TI MCP Integration Guide

Connect any MCP-compatible AI client to Transient Intelligence. The TI MCP server exposes a deterministic tool surface for upload, retrieval, and evidence-first querying. Full docs: [transientintelligence.com/docs/mcp-integration](https://transientintelligence.com/docs/mcp-integration)

## Setup

You can use TI MCP in two ways:
- Cloud MCP (recommended for onboarding): point client to hosted MCP URL and send API key in headers.
- Local stdio MCP (advanced/local dev): run local server process with env vars.

For a user-first onboarding path, see: [TI Claude Desktop Cloud Setup](claude-desktop-cloud-setup.md).

### Claude Desktop

Recommended: use **Settings -> Connectors -> Add custom connector**.

Connector values:
- URL: `https://api.transientintelligence.com/mcp`
- Header: `x-api-key: ti_int_replace_with_your_real_key`

Restart Claude Desktop after saving.

Config-file fallback (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "transient-intelligence": {
      "url": "https://api.transientintelligence.com/mcp",
      "headers": {
        "x-api-key": "ti_int_replace_with_your_real_key"
      }
    }
  }
}
```

Fallback for older Claude builds: use command-based local stdio mode (`node` + `ti-mcp-stdio.mjs` + `TI_API_KEY` env).

### Cursor

Config: `.cursor/mcp.json`

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

### ChatGPT Desktop

Config: `~/Library/Application Support/ChatGPT/mcp.json`. Can use `npx -y @transient-intelligence/mcp-server` with `TI_API_KEY` in env.

## Default workflow

Call `ti_workflow_policy` once at session start, then use `ti_run_workflow` for all upload-and-ask operations.

**Flow:** `ti_workflow_policy` → `ti_run_workflow` → Citations returned

### ti_run_workflow examples

**Inline text (fastest path):**
```
ti_run_workflow({ question: "What was the Q3 revenue?", text: "Q3 revenue reached $1.2M driven by enterprise signups.", top_k: 5 })
```

**HTTPS file URL:**
```
ti_run_workflow({ question: "Summarise the key findings.", input_url: "https://storage.example.com/report.pdf", filename: "report.pdf" })
```

**Reuse existing session:**
```
ti_run_workflow({ session_id: "ti-01-session-abc123", question: "What were the risks identified?" })
```

## Session management (important)

Reusing the same `session_id` across unrelated reviews can muddy prior context and make results harder to interpret.

- Reuse `session_id` only when follow-up questions refer to the same source set.
- Start a new session for a new document pack, a new review objective, or a new customer/case.
- For strict separation, treat each review job as a fresh session and persist the returned `session_id` per job.
- In multi-tenant workflows, never share a `session_id` across tenants/projects.

## Manual flow

For fine-grained control: `ti_upload` → `ti_run_status` → `ti_ask`

- **ti_upload** — Ingest extracted content, get `session_id` and optional `run_id`
- **ti_run_status** — Poll until `status: completed` before calling `ti_ask`
- **ti_ask** — Query once `retrieval_ready: true`

## Transport decision table

| What you have | Tool to use | Notes |
|---------------|-------------|-------|
| Extracted text / JSON | ti_upload or ti_run_workflow | Fastest path. Pass as `text` or `json_payload` |
| Signed / public HTTPS URL | ti_fetch_upload or ti_run_workflow | Server-side fetch. Preserves Excel sheet fidelity |
| Local mount path `/mnt/data` | ti_create_upload_handoff | Creates one-click browser upload URL. Do not pass local paths as text |
| Text + question (one call) | ti_run_workflow | Recommended default |
| Existing session_id | ti_ask | Query without re-uploading |

**Local paths:** In sandboxed clients (ChatGPT Desktop, Claude.ai web), paths like `/mnt/data/file.pdf` are not accessible. Use `ti_create_upload_handoff` to generate a browser upload URL.

## Response handling

- Always check `ok` before processing output
- If `ok` is false, surface `error` and `next_steps`
- If `retrieval_ready: false`, poll `ti_run_status` then retry
