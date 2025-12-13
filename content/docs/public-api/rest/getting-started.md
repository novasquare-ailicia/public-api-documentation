---
title: Getting Started
type: docs
weight: 50
prev: /docs/public-api/rest/endpoints
next: /docs/public-api/rest/events
---

## Steps
1) Get your API key (see [Authentication](/docs/public-api/authentication)).  
2) Add the header: `Authorization: Bearer <API_KEY>`.  
3) Choose: send context (`/v1/events`) or trigger reaction (`/v1/events/generations`).

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
      "content": "Jellab just landed a triple kill with Yasuo."
    }
  }'
'
```

{{< callout context="warning" title="Limits" >}}
Rate limit: 1 request per second per API key. Content limit: ~300 characters for `/generations`, ~700 characters for `/events`.
{{< /callout >}}

Next: consume outputs via [EventSub](/docs/public-api/eventsub).
