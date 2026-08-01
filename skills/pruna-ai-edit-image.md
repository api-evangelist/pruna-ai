---
name: Edit images with Pruna P-API
description: Upload reference images and run an image-edit generation against them.
api: openapi/pruna-ai-openapi.yml
operations: [uploadFile, createPrediction, getPredictionStatus, downloadContent]
---

# Edit images with the Pruna P-API

Use this skill to transform 1–5 reference images with the `p-image-edit` model.

## Auth
- Send the `apikey` header on every request.

## Steps

1. **Upload each reference image** — `uploadFile` (`POST /v1/files`), `multipart/form-data` with the file in the `content` field. Each response returns an `id` and a `urls.get` URL. Repeat for up to 5 images.
2. **Submit the edit** — `createPrediction` (`POST /v1/predictions`).
   - Header `Model: p-image-edit` (or `p-image-edit-lora`).
   - Body: `{ "input": { "prompt": "<edit instruction>", "images": ["<file get URL>", ...], "aspect_ratio": "16:9" } }` — pass the uploaded files' get URLs in `input.images`.
   - Optionally `Try-Sync: true` for an inline result.
3. **Poll** — `getPredictionStatus` (`GET /v1/predictions/status/{id}`) until `succeeded`; read `generation_url`.
4. **Download** — `downloadContent` (`GET /v1/predictions/delivery/{path}`) with the `apikey` header.

## Rules
- Uploaded files carry `expires_at`; upload shortly before you submit.
- Uploads are rate limited (HTTP 429) and size limited (HTTP 413) — back off on 429, shrink on 413.
- Same async/idempotency/error-envelope rules as the generate-image skill.
