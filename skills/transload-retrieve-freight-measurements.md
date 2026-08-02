---
name: transload-retrieve-freight-measurements
description: Retrieve computer-vision freight measurement results from the Transload Pipeline Backend API — list scans for a customer, follow a scan through its pipeline stages, and pull the latest AI measurement for a handling unit.
api: Transload Pipeline Backend API (https://api.transload.io)
generated: '2026-07-21'
method: generated
source: openapi/transload-pipeline-backend-openapi.json
operations:
  - GET /v1/scans
  - GET /v1/scans/{id}
  - GET /v1/scans/stats
  - GET /v1/handling-units/{handlingUnit}/latest-result
  - GET /v1/ai-results
  - GET /v1/customer/data/handling-units
---

# Retrieve freight measurements from Transload

Transload measures pallets and parcels with warehouse CCTV. Every measurement
flows through a scan pipeline (ingestion → dewarping → classification →
measurement → qa) and lands as an AI result attached to a handling unit.

The spec declares no operationIds, so operations are referenced by
`METHOD /path` exactly as published in the OpenAPI.

## Prerequisites

- A bearer token. All operations require `Authorization: Bearer <token>`
  (global `bearerAuth` security). Customer sessions come from the magic-link
  flow (`POST /v1/customer/auth/request-link` then `POST /v1/customer/auth/verify`);
  tokens can be refreshed with `POST /refresh-token`.
- Unauthenticated calls return `401` with the envelope
  `{code: "unauthorized", message, details, requestId}` — keep `requestId`
  for support.

## Steps

1. **List scans** — `GET /v1/scans?customerId=<uuid>` (`customerId` is
   required). Filter with `stage` (ingestion|dewarping|classification|
   measurement|qa), `stageStatus` (pending|running|succeeded|failed|skipped|
   rejected), and `capturedAfter`/`capturedBefore` (date-time). Page with
   `limit` (default 50) and `offset` (default 0).
2. **Inspect one scan** — `GET /v1/scans/{id}` for the scan's identifiers
   (handlingUnitId, shipmentId) and stage history.
3. **Get the measurement** — `GET /v1/handling-units/{handlingUnit}/latest-result`
   returns the latest AI measurement result for that handling unit;
   `GET /v1/ai-results` lists results in bulk.
4. **Aggregate** — `GET /v1/scans/stats` for scan volume statistics, or, as a
   customer session, `GET /v1/customer/data/handling-units` (and
   `/export`) for the customer-facing handling-unit dataset.

## Conventions

- Pagination is limit/offset; there is no idempotency-key support; errors use
  the `{code, message, details, requestId}` envelope
  (see `conventions/transload-conventions.yml` and
  `errors/transload-problem-types.yml`).
