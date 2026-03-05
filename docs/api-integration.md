# TI API Integration Guide

Contract-first reference for integrating TI via REST.

## Base URL and Auth

- Base URL: `https://api.transientintelligence.com`
- Auth header: `x-api-key: <YOUR_TI_API_KEY>`
- [Create account](https://transientintelligence.com/auth/register) · [Log in](https://transientintelligence.com/auth/login) · [Get API key](https://transientintelligence.com/dashboard/developers)

## Recommended default: `/api/models/v1/answer`

Use this endpoint for most integrations. It accepts inline input and handles upload/index/retrieval internally.

Required fields:

- `question` (string)
- One of: `input`, `input_url`, or `session_id`

Useful options:

- `top_k` (1-50, default 10)
- `wait_max_ms` (max wait for async indexing)
- `retry_after_seconds` (retry hint when indexing is in progress)

## Decoupled flow: `/upload` then `/ask`

Use this for large files or explicit async control.

1. `POST /api/models/v1/upload`
2. If `retrieval_ready: false`, poll `GET /api/models/v1/runs/{id}`
3. Once complete, call `POST /api/models/v1/ask`

## Supported input types

- `.pdf`
- `.docx`
- `.xlsx`
- `.csv`
- `.txt`
- `.json`

## Reliability guardrails

- Check `citations` before using answer content.
- Handle `action_required: wait_then_retry` explicitly.
- Tune `top_k`: 1-5 precision, 6-15 balanced, 16-50 recall.
