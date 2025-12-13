---
title: REST API Overview
type: docs
weight: 40
prev: /docs/public-api/openapi
next: /docs/public-api/rest/endpoints
---

Use the REST API to push contextual events to ai_licia or trigger a reaction. This is the write surface; EventSub is the read surface for real-time output.

| Action | Endpoint | Notes |
| --- | --- | --- |
| Send context | `POST /v1/events` | Up to ~700 chars |
| Trigger reaction | `POST /v1/events/generations` | Up to ~300 chars, 1 req/sec per API key |

Authentication:
```shell
Authorization: Bearer <API_KEY>
```

Next: [Endpoints summary](/docs/public-api/rest/endpoints), then [REST Getting Started](/docs/public-api/rest/getting-started) or [EventSub](/docs/public-api/eventsub) to consume outputs.
