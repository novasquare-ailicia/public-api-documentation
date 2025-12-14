---
title: Endpoints (summary)
type: docs
weight: 45
prev: /docs/public-api/rest
next: /docs/public-api/rest/getting-started
description: Quick matrix of available REST endpoints, their purpose, limits, and deep links into detailed docs.
---

## Summary

| Endpoint | Purpose | Limits | Details |
| --- | --- | --- | --- |
| `POST /v1/events` | Send contextual data | ~700 chars | [Docs](/docs/public-api/rest/events) |
| `POST /v1/events/generations` | Trigger AI reaction | ~300 chars; 1 request every ~25s per API key | [Docs](/docs/public-api/rest/events#post-v1-events-generations) |
| `GET /v1/characters` | List owned characters (id, name, description, active flag) | N/A | [Docs](/docs/public-api/rest/characters#get-v1characters) |
| `PUT /v1/characters/{characterId}/active` | Change the active character | Character must belong to the API key owner | [Docs](/docs/public-api/rest/characters#put-v1characterscharacteridactive) |
| `POST /v1/streams/{channelName}` | Request ai_licia to join chat | Requires channel authorization + active subscription/credits | [Docs](/docs/public-api/rest/streams#post-v1streamschannelname) |
| `DELETE /v1/streams/{channelName}` | Request ai_licia to leave chat | Requires channel authorization | [Docs](/docs/public-api/rest/streams#delete-v1streamschannelname) |
