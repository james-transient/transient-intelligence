# TI Claude Desktop Cloud Setup

Use this guide to connect Claude Desktop to the hosted TI MCP endpoint.

## Important compatibility note

Claude Desktop MCP UX has changed in recent versions.  
Primary path is now **Settings -> Connectors -> Add custom connector**.

There is currently no published one-click TI Desktop Extension (`.mcpb`) install link, so use Connector/manual setup for now.

If your build does not expose custom headers in Connector UI, use the config-file fallback in this guide.

## Quick checklist (about 3 minutes)

- Get your TI API key.
- Open Claude Desktop Connectors.
- Add TI cloud MCP URL and API key header.
- Restart Claude Desktop.
- Run one TI tool call to confirm.

## 1) Get your TI API key

- [Create account](https://transientintelligence.com/auth/register)
- [Log in](https://transientintelligence.com/auth/login)
- [Developers dashboard](https://transientintelligence.com/dashboard/developers)

Use a safe placeholder while editing docs and examples:

`ti_int_replace_with_your_real_key`

## 2) Add connector in Claude Desktop (recommended)

In Claude Desktop:

1. Open **Settings**
2. Open **Connectors**
3. Click **Add custom connector**
4. Set MCP URL to:
   - `https://api.transientintelligence.com/mcp`
5. Add header:
   - `x-api-key: ti_int_replace_with_your_real_key`
6. Save connector and restart Claude

## 3) Config-file fallback (older/advanced builds)

If Connector UI is unavailable or missing custom header support, edit:

`~/Library/Application Support/Claude/claude_desktop_config.json`

Optional backup:

```bash
cp "$HOME/Library/Application Support/Claude/claude_desktop_config.json" \
  "$HOME/Library/Application Support/Claude/claude_desktop_config.json.backup.$(date +%Y%m%d-%H%M%S)" 2>/dev/null || true
```

Then add:

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

Notes:
- `x-api-key` is required.
- Never commit real keys to git or screenshots.
- This is cloud setup, so you do not need a local TI repo path.

## 4) Restart Claude Desktop

Fully quit Claude Desktop, then reopen.

## 5) Verify setup

Run these in Claude:

```text
ti_workflow_policy()
ti_api_key_status()
ti_run_workflow({ question: "What is TI?", text: "Transient Intelligence is evidence-first..." })
```

If tools respond, setup is complete.

Session scope note:
- `session_id` is review-scope state, not user-scope state.
- Reuse a session only for follow-up questions on the same evidence set.
- Start a new session for new documents or review goals.

## 6) If tools do not appear

- Confirm JSON format is valid.
- Confirm MCP URL is `https://api.transientintelligence.com/mcp`.
- Confirm key starts with `ti_int_`.
- Restart Claude Desktop again after any config edit.

If your Claude Desktop build does not support `url` MCP entries, use the MCP integration guide fallback in `docs/mcp-integration.md` for command-based bridge mode.
