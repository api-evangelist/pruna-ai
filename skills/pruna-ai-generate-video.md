---
name: Generate a video with Pruna P-API
description: Submit a text/image-to-video generation using the async workflow and download the clip.
api: openapi/pruna-ai-openapi.yml
operations: [createPrediction, getPredictionStatus, downloadContent]
---

# Generate a video with the Pruna P-API

Use this skill for text-to-video / image-to-video generation. Video jobs are
longer, so always use the asynchronous (poll) workflow — do not rely on
`Try-Sync`, which only waits up to 60 seconds.

## Auth
- Send the `apikey` header on every request.

## Steps

1. **Submit** — `createPrediction` (`POST /v1/predictions`).
   - Header `Model:` a video model, e.g. `p-video`, `wan-t2v` (text-to-video), or `wan-i2v` (image-to-video).
   - Body: `{ "input": { "prompt": "<scene>", ... } }`. For image-to-video, first upload a source image (see the edit-image skill) and pass its get URL in `input`.
   - Do NOT set `Try-Sync` — submit and poll.
2. **Poll** — `getPredictionStatus` (`GET /v1/predictions/status/{id}`) until `status` is `succeeded`; read `generation_url`.
3. **Download** — `downloadContent` (`GET /v1/predictions/delivery/{path}`) with the `apikey` header; save the MP4.

## Rules
- Poll on a sensible interval; video jobs run longer than images.
- Non-idempotent submits; handle the `{ "error": { code, message } }` envelope (`TIMEOUT`, `MODEL_ERROR`, `QUOTA_EXCEEDED`).
- Delivered content expires (HTTP 410) — download promptly.
