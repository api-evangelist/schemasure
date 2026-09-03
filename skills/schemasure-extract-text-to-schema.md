---
name: schemasure-extract-text-to-schema
description: Turn unstructured text or HTML into JSON guaranteed to validate against your JSON Schema, paying $0.01 in USDC per successful call via x402 — or nothing on failure.
api: SchemaSure Structured Extraction API
operations:
  - extract_to_schema
generated: '2026-09-03'
method: generated
source: openapi/schemasure-openapi.json + llms/schemasure-llms.txt
---

# Extract text or HTML to schema-valid JSON

1. Write a JSON Schema (draft 2020-12) for the output you want. Accuracy levers the provider documents: add a `description` to every ambiguous field ("tax rate as a decimal such as 0.0725"), pin values with `enum`/`type`/`format`, and make possibly-absent fields nullable so you get `null` instead of a guess. Keep `options.strict` true (default) to drop fields not in your schema.
2. POST to `https://schemasure.com/v2/extract` (operationId `extract_to_schema`) with body `{"input": <text or HTML, max 256 KB>, "schema": <your JSON Schema>, "inputType": "auto"}`.
3. The unsigned request returns HTTP 402. Decode the `PAYMENT-REQUIRED` header and validate the live price, network (`eip155:8453`), asset (USDC), `payTo`, and `resource` — never hardcode payment terms.
4. Sign those terms with an x402 V2-compatible wallet and retry the identical body with the `PAYMENT-SIGNATURE` header.
5. On 200, `data` is guaranteed to validate against your schema; `meta` reports `repairs` and `latencyMs`. Decode `PAYMENT-RESPONSE` for the settlement receipt.
6. On error, the body is `{"error":{"code","message"}}` and the call is not charged. 400-class codes (`BAD_REQUEST`, `INVALID_SCHEMA`, `INPUT_TOO_LARGE`, `UNSUPPORTED_INPUT_TYPE`): fix the request, do not retry unchanged. 422 (`EXTRACTION_FAILED`, `VALIDATION_FAILED`, `ABSTAINED`): change input/schema or fall back. 429 (`RATE_LIMITED`): wait the `Retry-After` duration. 5xx: bounded exponential backoff.
7. Payment idempotency is NOT offered — never blindly retry a paid request after losing its response.

To evaluate free first: POST the same body to legacy `https://schemasure.com/extract` (operationId `extract_to_schema_v1`) — 3 free calls per client, text/HTML only.
