---
title: POST /v1/events
type: docs
weight: 60
prev: /docs/public-api/rest/getting-started
next: /docs/public-api/rest/generations
---

Send contextual data so ai_licia can react with awareness.

**Request**
- Method/Path: `POST /v1/events`
- Headers: `Authorization: Bearer <API_KEY>`, `Content-Type: application/json`
- Body:
  - `eventType` (string, required): e.g. `GAME_EVENT`
  - `data.channelName` (string, required): channel name
  - `data.content` (string, required, max ~700 chars): contextual payload

**Responses**
- `200 OK` on success
- `401 Unauthorized` if the key is missing/invalid
- `422 Unprocessable Entity` if content exceeds the limit
Errors surface as JSON with status/message.

**Example (curl)**
```bash
curl -X POST https://api.getailicia.com/v1/events \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "GAME_EVENT",
    "data": {
      "channelName": "mychannel",
      "content": "Altitude 18 ft, heading 304, speed 5 knots"
    }
  }'
'
```
