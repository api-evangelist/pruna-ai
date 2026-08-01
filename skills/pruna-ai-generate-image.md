---
name: Generate an image with Pruna P-API
description: Submit a text-to-image generation, poll for completion, and download the result.
api: openapi/pruna-ai-openapi.yml
operations: [createPrediction, getPredictionStatus, downloadContent]
---

# Generate an image with the Pruna P-API

Use this skill to turn a text prompt into an image using a Pruna P-Series model.

## Auth
- Every request needs the `apikey` header with your Pruna API key (from the dashboard at https://dashboard.pruna.ai/login).

## Steps

1. **Submit the generation** — `createPrediction` (`POST /v1/predictions`).
   - Set the `Model` header to a text-to-image model, e.g. `p-image`, `flux-dev`, or `qwen-image`.
   - Body: `{ "input": { "prompt": "<your prompt>", "aspect_ratio": "16:9" } }`.
   - Optionally set `Try-Sync: true` to block up to 60s and get the finished result inline (skip step 2).
2. **Poll status** — `getPredictionStatus` (`GET /v1/predictions/status/{id}`) using the `id` from step 1. Repeat until `status` is `succeeded` (or handle `failed`/`canceled`). Read `generation_url` from the response.
3. **Download** — `downloadContent` (`GET /v1/predictions/delivery/{path}`) at the `generation_url`, still sending the `apikey` header. Save the returned image bytes.

## Rules
- Async by default: submit, then poll. Only expect an inline result when you sent `Try-Sync: true`.
- Not idempotent — each `createPrediction` enqueues a new billable generation; do not blindly retry a submit.
- Handle errors from the `{ "error": { "code", "message" } }` envelope; `QUOTA_EXCEEDED` means you are out of credits, `INVALID_INPUT` means fix the prompt/params.
- Delivered content expires (HTTP 410) — download promptly.
