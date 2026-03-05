# TI MCP Integration Guide

Connect any MCP-compatible AI client to Transient Intelligence. The TI MCP server exposes a deterministic tool surface for upload, retrieval, and evidence-first querying. Full docs: [transientintelligence.com/docs/mcp-integration](https://transientintelligence.com/docs/mcp-integration)

## Setup

The TI MCP server runs as a local stdio process. Add it to your client's MCP configuration and provide your API key via the `TI_API_KEY` environment variable.

### Claude Desktop

Config: `~/Library/Application Support/Claude/claude_desktop_config.json`. Restart Claude Desktop after saving.

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
