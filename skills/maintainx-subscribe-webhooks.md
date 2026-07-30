---
name: Subscribe to MaintainX webhooks and verify payloads
description: Create a webhook subscription for an event type, retrieve its signing secret, and manage the subscription.
api: openapi/maintainx-openapi-original.json
operations:
  - POST /subscriptions
  - GET /subscriptions/{id}
  - GET /subscriptions/{id}/secret
  - PATCH /subscriptions/{id}
  - DELETE /subscriptions/{id}
---

# Subscribe to MaintainX webhooks and verify payloads

Operations are cited by METHOD + path (the source OpenAPI declares no `operationId`s).
Base URL `https://api.getmaintainx.com/v1`; send `Authorization: bearer {token}`.

## Steps
1. Create a subscription — `POST /subscriptions` with `{ "type": "<EVENT>", "url": "https://your.endpoint" }`.
   `<EVENT>` is one of the 46 documented types (e.g. `NEW_WORK_ORDER`, `WORK_ORDER_STATUS_CHANGE`,
   `ASSET_STATUS_CHANGE`). Some events accept a `filters` object. See
   `asyncapi/maintainx-webhooks.yml` for the full catalog.
2. Fetch the signing secret — `GET /subscriptions/{id}/secret` — and store it to verify the HMAC
   signature on every inbound POST.
3. Update the target URL or filters — `PATCH /subscriptions/{id}`.
4. Inspect a subscription — `GET /subscriptions/{id}`.
5. Remove it — `DELETE /subscriptions/{id}`.

## Rules
- One subscription targets one event type; create several for multiple events.
- Verify inbound payloads with the signing secret before trusting them.
- MaintainX recommends webhooks over polling for real-time updates. Write operations accept a
  `skipWebhook` query param to suppress emission when you do not want to trigger your own hooks.
- Respect rate limits (`rate-limits/maintainx-rate-limits.yml`); errors use the
  `{ error } / { errors }` envelope (`errors/maintainx-problem-types.yml`).
