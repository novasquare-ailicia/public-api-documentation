---
title: Getting Started
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
| List callable Recipes | `GET /v1/recipes` | [Setup and requirements](/docs/public-api/rest/recipes) |
| Trigger a Recipe | `POST /v1/recipes/{recipeId}/runs` | Typed inputs; connected desktop required |
| Recipe webhook | `POST /v1/recipe-webhooks/{webhookId}` | Dedicated limited token |
| Send context | `POST /v1/events` | Up to 1,000 chars |
| Trigger reaction | `POST /v1/events/generations` | Up to 300 chars; independent per-key windows: `STANDARD` ~25s, `FAST` 1s |
| List characters | `GET /v1/characters` | Returns id, name, description, active state |
| Set active character | `PUT /v1/characters/{characterId}/active` | Only works for owned characters |
| Channel status | `GET /v1/channels/status` | Returns `channelName`, `isPrimary`, `isLive`, `isAiInChat` |
| Join chat | `POST /v1/streams/{channelName}` | Waits up to 10s for ai_licia to join (`mode=LIVE|TEST`) |
| Leave chat | `DELETE /v1/streams/{channelName}` | Sends an async leave request |

Authentication:
```shell
Authorization: Bearer <API_KEY>
```

Next: [Endpoints summary](/docs/public-api/rest/endpoints), then [REST Getting Started](/docs/public-api/rest/getting-started) or [EventSub](/docs/public-api/eventsub) to consume outputs.
