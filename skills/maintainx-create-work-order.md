---
name: Create and track a MaintainX work order
description: Create a work order, attach it to an asset, poll for status changes, and add a comment.
api: openapi/maintainx-openapi-original.json
operations:
  - POST /workorders
  - GET /workorders/{id}
  - GET /workorders
  - POST /workorders/{id}/comments
  - GET /assets/{id}
---

# Create and track a MaintainX work order

MaintainX operations are referenced by METHOD + path — the source OpenAPI declares no
`operationId`s, so every step below cites the verified method/path from
`openapi/maintainx-openapi-original.json`.

## Auth
Send `Authorization: bearer {token}` on every request. Generate the token in-app at
Settings > Integrations > API Keys. See `authentication/maintainx-authentication.yml`.
Base URL: `https://api.getmaintainx.com/v1`.

## Steps
1. (Optional) Confirm the target asset exists — `GET /assets/{id}`.
2. Create the work order — `POST /workorders` with the title, description, priority, and the
   asset/location association. Capture the returned `id`.
3. Read it back — `GET /workorders/{id}`.
4. Add a comment / update — `POST /workorders/{id}/comments`.
5. Track changes — poll `GET /workorders` with `updatedAt[gte]=<lastSync>`, carrying the filter
   across every page (Work Orders use **cursor** pagination — follow the `cursor`), then advance
   your watermark to the max `updatedAt`. Prefer the `WORK_ORDER_STATUS_CHANGE` webhook over polling.

## Rules
- Rate limits: 100 req/60s per user, 500 req/60s per org. Watch `X-Rate-Limit-Remaining`; on
  `429` wait `X-Rate-Limit-Reset` seconds plus a buffer (`rate-limits/maintainx-rate-limits.yml`).
- No idempotency key — do not blindly retry `POST`; check via `GET` before re-creating
  (`conventions/maintainx-conventions.yml`).
- Errors arrive as `{ "error": ... }` or `{ "errors": [...] }` (`errors/maintainx-problem-types.yml`).
