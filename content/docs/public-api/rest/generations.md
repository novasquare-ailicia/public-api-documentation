---
title: POST /v1/events/generations
type: docs
weight: 70
prev: /docs/public-api/rest/events
next: /docs/public-api/eventsub
---

Trigger ai_licia to react immediately.

**Request**
- Method/Path: `POST /v1/events/generations`
- Headers: `Authorization: Bearer <API_KEY>`, `Content-Type: application/json`
- Body:
  - `eventType` (string, required): e.g. `GAME_EVENT`
  - `data.channelName` (string, required): channel name
  - `data.content` (string, required, max ~300 chars): what to react to

**Responses**
- `200 OK` on success
- `401 Unauthorized` if the key is missing/invalid
- `422 Unprocessable Entity` if content exceeds the limit
- `429 Too Many Requests` if you exceed 1 req/sec per API key
Errors surface as JSON with status/message.

**Example (curl)**
```bash
curl -X POST https://api.getailicia.com/v1/events/generations \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "GAME_EVENT",
    "data": {
      "channelName": "mychannel",
      "content": "Player just cleared the boss with 1 HP left."
    }
  }'
'
```
