---
title: REST API Overview
type: docs
weight: 40
prev: /docs/public-api/openapi
next: /docs/public-api/rest/endpoints
description: Feed ai_licia contextual events and on-demand reactions through the REST endpoints.
summary: REST entry point covering context ingestion, instant reactions, authentication, and usage patterns.
---

Use the REST API to push contextual events to ai_licia or trigger a reaction.

| Action | Endpoint | Notes |
| --- | --- | --- |
| Send context | `POST /v1/events` | Up to ~700 chars |
| Trigger reaction | `POST /v1/events/generations` | Up to ~300 chars, 1 request every ~25s per API key |
| List characters | `GET /v1/characters` | Returns id, name, description for the API key owner |
| Set active character | `PUT /v1/characters/{characterId}/active` | Only works for owned characters |
| Join chat | `POST /v1/streams/{channelName}` | Waits up to 10s for ai_licia to join |
| Leave chat | `DELETE /v1/streams/{channelName}` | Sends an async leave request |

Authentication:
```shell
Authorization: Bearer <API_KEY>
```

Next: [Endpoints summary](/docs/public-api/rest/endpoints), then [REST Getting Started](/docs/public-api/rest/getting-started) or [EventSub](/docs/public-api/eventsub) to consume outputs.
