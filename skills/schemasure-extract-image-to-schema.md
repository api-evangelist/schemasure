---
name: schemasure-extract-image-to-schema
description: Turn a document image (invoice, receipt, form, screenshot) into JSON guaranteed to validate against your JSON Schema — $0.03 in USDC per successful image via x402, failed calls free.
api: SchemaSure Structured Extraction API
operations:
  - extract_image_to_schema
generated: '2026-09-03'
method: generated
source: openapi/schemasure-openapi.json + llms/schemasure-llms.txt
---

# Extract a document image to schema-valid JSON

1. Prepare the image: PNG, JPEG, or WebP as raw base64 bytes (no `data:` URL prefix), max 8 MiB decoded. Clear, upright, high-resolution images work best. PDFs are not supported.
2. Write a JSON Schema (draft 2020-12) for the fields you want, with per-field `description`s for anything ambiguous ("Invoice identifier exactly as printed", "Final amount due as a number, without currency symbols").
3. POST to `https://schemasure.com/v2/extract-image` (operationId `extract_image_to_schema`) with body `{"image":{"data":"<base64>","mimeType":"image/png"},"schema":<your JSON Schema>}`.
4. This endpoint is always paid — there is intentionally no free or V1 image route. Handle the 402 `PAYMENT-REQUIRED` challenge exactly as for text: validate the live terms, sign with an x402 V2 wallet, retry with `PAYMENT-SIGNATURE`.
5. On 200, `data` validates against your schema (`meta.validated` is always true); on any error you are not charged. The server validates base64, size, and file signature before model work, so bad images fail fast as `BAD_REQUEST`/`UNSUPPORTED_INPUT_TYPE`.
6. Same retry discipline as text: fix 400s, adapt on 422s, honor `Retry-After` on 429, back off on 5xx, and never blindly retry a paid request after losing its response (no payment idempotency).
