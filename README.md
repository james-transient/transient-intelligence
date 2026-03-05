# Solving AI Hallucination: TI

Transient Intelligence (TI) is a developer-facing reliability layer for document-heavy AI workflows, designed to reduce hallucination risk with evidence ranking and citation-grounded outputs.

## What This Repository Includes

- Public API-oriented documentation and usage patterns
- Integration guidance for developer workflows
- Security and contribution policy files for public collaboration

## What TI Solves

- Reduces unsupported claims by prioritizing ranked evidence
- Improves traceability with citation-grounded response patterns
- Supports production integration using clear API contracts

## Quickstart (API Contract Level)

1. Obtain API credentials.
2. Review the OpenAPI contract and example requests.
3. Send analysis requests with your document payloads.
4. Consume ranked evidence and citations in responses.

Example request shape:

```json
{
  "input": {
    "text": "Document content to analyze"
  },
  "options": {
    "mode": "analysis"
  }
}
```

## Public-Safe Scope

This repository is intentionally limited to public-safe, developer-facing materials. Proprietary model internals, private infrastructure details, and undisclosed algorithms are not included.

## Related Project

- Transient Recall (TR): [transient-recall](https://github.com/james-transient/transient-recall)
