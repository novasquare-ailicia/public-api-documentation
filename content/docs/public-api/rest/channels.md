---
title: Channel Runtime Status
type: docs
weight: 75
prev: /docs/public-api/rest/characters
next: /docs/public-api/rest/streams
description: Use GET /v1/channels/status to fetch runtime status for channels linked to your API key.
---

Use this endpoint to read runtime status for channels controlled by the authenticated API key owner.

Each item contains only:
- `channelName`
- `isPrimary`
- `isLive`
- `isAiInChat`

## GET /v1/channels/status

**Request**
- Method/Path: `GET /v1/channels/status`
- Headers: `Authorization: Bearer <API_KEY>`

**Responses**
- `200 OK` with an array of channel status objects
- `401 Unauthorized` if the API key is invalid

**Example**

```bash
curl https://api.getailicia.com/v1/channels/status \
  -H "Authorization: Bearer ${API_KEY}"
```

```json
[
  {
    "channelName": "mychannel",
    "isPrimary": true,
    "isLive": true,
    "isAiInChat": true
  },
  {
    "channelName": "mysecondchannel",
    "isPrimary": false,
    "isLive": false,
    "isAiInChat": false
  }
]
```
