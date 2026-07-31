---
name: Export AI Assistant conversation and interaction data
description: Page the Moveworks Data API to export AI Assistant conversations, interactions, plugin calls, plugin resources, and users into an analytics store.
api: openapi/moveworks-servicenow-data-api-openapi.yml
operations:
  - listConversations
  - listInteractions
  - listPluginCalls
  - listPluginResources
  - listUsers
---

# Export Moveworks AI Assistant data

Use the read-only Moveworks Data API to export interaction data for analytics
(the shape the reference `moveworks/data-api` repo loads into Snowflake / PowerBI).

## Authentication
- Send `Authorization: Bearer <token>` on every request.
- Prefer an OAuth 2.0 client-credentials token (Client ID + Secret from the dashboard,
  exchanged at the OAuth token endpoint; access tokens expire after 60 seconds), or a
  long-lived API key. Use a dedicated service account, not a personal account.
- Pick the base host for your region, e.g. `https://api.moveworks.ai/export/v1` (US).

## Steps
1. Call `listConversations` to fetch top-level conversation groupings.
2. For coverage of every exchange, call `listInteractions` (utterances, button/link
   clicks, form submissions).
3. Call `listPluginCalls` then `listPluginResources` to capture plugin invocations
   and the resources they touched.
4. Call `listUsers` to fetch the user roster referenced by the records.
5. For each call, read the OData envelope: iterate `value[]`, then follow
   `odata.nextlink` and repeat until it is absent. Pass `$count=true` when you need a
   total.

## Rules and conventions
- Read-only: all operations are GET; there is no idempotency key because nothing is written.
- Data retention is 30 days and new data has up to a 24-hour availability SLA — schedule
  incremental pulls accordingly.
- On `401` refresh the token (OAuth tokens live 60 seconds); on `429` back off and retry;
  a `410 Gone` means the API version was sunset — move to the current stable `v1`.
- Errors return `{ "error": { "code": ..., "message": ... } }`; error codes are stable.
