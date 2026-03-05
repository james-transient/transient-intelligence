# Transient Intelligence Quickstart

This quickstart mirrors the public TI docs and gets you to your first grounded answer fast.

## 1) Get an API key

All requests require the `x-api-key` header.

- [Create account](https://transientintelligence.com/auth/register)
- [Log in](https://transientintelligence.com/auth/login)
- [Generate API key](https://transientintelligence.com/dashboard/developers)

```text
x-api-key: <YOUR_TI_API_KEY>
```

## 2) Make your first request

Use the production base URL:

```text
https://api.transientintelligence.com
```

```bash
curl -X POST "https://api.transientintelligence.com/api/models/v1/answer" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question":"What are the key findings?","input":"Document text here...","top_k":5}'
```

## 3) Validate response discipline

Treat citations as the ground truth.

- If `citations.length > 0`, render grounded results.
- If `citations` is empty, return "No evidence found".
- Do not fill gaps with unsupported model assumptions.

## 4) Choose your integration path

- Default: `POST /api/models/v1/answer`
- Advanced async flow: `POST /upload` -> poll `GET /runs/{id}` -> `POST /ask`

See:

- [API integration guide](api-integration.md)
- [MCP integration guide](mcp-integration.md)
