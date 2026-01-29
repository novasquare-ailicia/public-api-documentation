---
title: Getting Started
type: docs
weight: 50
prev: /docs/public-api/rest/endpoints
next: /docs/public-api/rest/events
description: Step-by-step guide for authenticating, sending your first context event, triggering generations, and managing chat presence via REST.
---

## Steps
1) Get your API key (see [Authentication](/docs/public-api/authentication)).  
2) Add the header: `Authorization: Bearer <API_KEY>`.  
3) Choose an action: send context (`/v1/events`), trigger a reaction (`/v1/events/generations`), list/activate characters, or have ai_licia join/leave your chat.

## First request (ingest an event)

```bash
curl -X POST https://api.getailicia.com/v1/events \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "GAME_EVENT",
    "data": {
      "channelName": "mychannel",
      "content": "Player reached level 20 with 2 HP remaining."
    }
  }'
'
```

Expected responses: `200 OK` success; `401` auth failure; `422` length issues.

## Trigger a reaction (rate limited)

```bash
curl -X POST https://api.getailicia.com/v1/events/generations \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "GAME_EVENT",
    "data": {
      "channelName": "mychannel",
      "content": "Jellab just landed a triple kill with Yasuo.",
      "features": {
        "vision": false,
        "web_search": false,
        "commands": false,
        "memory": false,
        "tts": true
      }
    }
  }'
'
```

{{< callout context="warning" title="Limits" >}}
Rate limit: 1 request every ~25 seconds per API key. Content limit: ~300 characters for `/generations`, ~700 characters for `/events`.
{{< /callout >}}

## Manage characters & chat presence

- List available personas: `GET /v1/characters`
- Activate one: `PUT /v1/characters/{characterId}/active`
- Make ai_licia join chat: `POST /v1/streams/{channelName}`
- Ask her to leave: `DELETE /v1/streams/{channelName}`

See the dedicated docs for payloads/responses: [Send Context & Trigger Reactions](/docs/public-api/rest/events), [Character Management](/docs/public-api/rest/characters), and [Control Chat Presence](/docs/public-api/rest/streams).

Next: consume outputs via [EventSub](/docs/public-api/eventsub).
