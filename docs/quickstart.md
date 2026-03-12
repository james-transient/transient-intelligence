# Transient Intelligence — Quick start

Get your API key and complete your first successful analysis via the Transient Intelligence API. Full docs: [transientintelligence.com/docs/developer-quickstart](https://transientintelligence.com/docs/developer-quickstart)

## 1. Get Your API Key

Every request requires an API key in the `x-api-key` header. Create an account, then generate a key from your developer dashboard.

- [Create account](https://transientintelligence.com/auth/register)
- [Log in](https://transientintelligence.com/auth/login)
- [Generate API key](https://transientintelligence.com/dashboard/developers)

**Keep it safe.** Your API key is tied to your account credits. Never commit it to version control or expose it client-side.

## 2. Your First Request

The simplest way to use the API is the one-call orchestrator endpoint. It handles upload, processing, and query answering in a single request.

**Production endpoint:** `https://api.transientintelligence.com`

### cURL

```bash
curl -X POST "https://api.transientintelligence.com/api/models/v1/answer" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What was the Q3 revenue?",
    "input": "The Q3 revenue reached $1.2M, driven by new enterprise signups.",
    "top_k": 3
  }'
```

### JavaScript

```javascript
const response = await fetch("https://api.transientintelligence.com/api/models/v1/answer", {
  method: "POST",
  headers: {
    "x-api-key": "YOUR_API_KEY",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    question: "What was the Q3 revenue?",
    input: "The Q3 revenue reached $1.2M, driven by new enterprise signups.",
    top_k: 3
  })
});
const data = await response.json();
console.log(data.citations);
```

### Python

```python
import requests

response = requests.post(
  "https://api.transientintelligence.com/api/models/v1/answer",
  headers={
    "x-api-key": "YOUR_API_KEY",
    "Content-Type": "application/json"
  },
  json={
    "question": "What was the Q3 revenue?",
    "input": "The Q3 revenue reached $1.2M, driven by new enterprise signups.",
    "top_k": 3
  }
)
data = response.json()
print(data["citations"])
```

## 3. Response shape (evidence-backed answer)

```json
{
  "success": true,
  "sessionId": "session_abc123",
  "citations": [
    {
      "doc_id": "inline_input",
      "snippet": "The Q3 revenue reached $1.2M",
      "locator": "paragraph 1"
    }
  ],
  "factual_confidence": {
    "label": "Strong",
    "message": "Answer is fully supported by evidence."
  }
}
```

## 4. Validate response discipline

- If `citations.length > 0`, render grounded results.
- If `citations` is empty, return "No evidence found" — do not fall back to `answer.summary`.
- Do not fill gaps with unsupported model assumptions.

## 5. Session management (important)

- Treat `session_id` as review-scope state, not user-scope state.
- Reuse a session only for follow-up questions on the same evidence set.
- Start a new session when documents, review goals, or customer/case context changes.
- Persist `session_id` per job/workflow, not as a global shared default.

## 6. Let AI build your integration

Copy this prompt and paste it into Claude, ChatGPT, or any capable model. Fill in the placeholder at the bottom and it will generate working integration code.

**Never paste your real API key into an AI chat.** Use `TI_API_KEY` environment variable as a placeholder.

```
You are helping me integrate the Transient (TI) API into my project.

## What TI does
TI is an evidence-first document analysis API. You send a question and document text; it returns a cited answer grounded only in the document — no hallucination.

## API contract
Endpoint:  POST https://api.transientintelligence.com/api/models/v1/answer
Auth:      x-api-key request header — value read from TI_API_KEY environment variable
Body:      { question: string, input: string, top_k?: number }
Response:  { answer: { summary, confidence }, citations: [{ quote, locator, relevance }], run_id }

## Rules to follow in the code you generate
- Read the API key from the TI_API_KEY environment variable — never hardcode it
- Always check citations.length > 0 before using the answer
- If citations is empty, surface 'No evidence found' — do not fall back to answer.summary
- Pass document text as input, never a file path
- Set Content-Type: application/json on every request

## My integration target
[Replace this line: describe your stack, platform, and use case — e.g.
 'n8n workflow triggered by a new Google Drive file, post cited results to Slack'
 'Python Flask endpoint that accepts a PDF upload and returns a cited summary'
 'Make scenario: Notion page updated → analyse → write answer back to Notion']
```

## 7. SDKs & client libraries

There is currently no official SDK. The API is plain REST — any HTTP client works. To auto-generate a typed client, download the [OpenAPI YAML](https://transientintelligence.com/TI01_V1_OPENAPI.yaml) from the docs site and use [openapi-generator](https://openapi-generator.tech) or [openapi-ts](https://github.com/hey-api/openapi-ts).

## Next steps

- [API integration guide](api-integration.md)
- [MCP integration guide](mcp-integration.md)
- [Claude Desktop cloud setup](claude-desktop-cloud-setup.md)
- [AI Client setup guides](https://transientintelligence.com/docs/integrations/ai-clients)
- [Automation platform guides](https://transientintelligence.com/docs/integrations/automation)
