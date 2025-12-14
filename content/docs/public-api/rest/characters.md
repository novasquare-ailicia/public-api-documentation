---
title: Character Management
type: docs
weight: 75
prev: /docs/public-api/rest/events
next: /docs/public-api/rest/streams
description: Learn how to list ai_licia characters and switch the active persona via the public REST API.
---

Use these endpoints to inspect the characters tied to your API key and switch the active persona.

## GET /v1/characters

Returns the list of characters owned by the authenticated streamer. Each item contains the identifier, display name, public description, and whether it's currently the active persona.

**Request**
- Method/Path: `GET /v1/characters`
- Headers: `Authorization: Bearer <API_KEY>`

**Responses**
- `200 OK` with an array of summaries
- `401 Unauthorized` if the key is missing/invalid

**Example**

```bash
curl -X GET https://api.getailicia.com/v1/characters \
  -H "Authorization: Bearer ${API_KEY}"
```

```json
[
  {
    "id": "char_123",
    "name": "Captain Vibes",
    "description": "Energetic persona that hypes the chat.",
    "isActive": true
  },
  {
    "id": "char_456",
    "name": "Cozy Willow",
    "description": "A calm storyteller for late-night sessions.",
    "isActive": false
  }
]
```

## PUT /v1/characters/{characterId}/active

Marks a character as the active persona for the streamer who owns the supplied API key.

**Request**
- Method/Path: `PUT /v1/characters/{characterId}/active`
- Headers: `Authorization: Bearer <API_KEY>`
- Path parameters:
  - `characterId`: identifier from `GET /v1/characters`

**Responses**
- `204 No Content` when the character becomes active
- `401 Unauthorized` if the key is missing/invalid
- `403 Forbidden` if the character belongs to another streamer
- `404 Not Found` if the characterId does not exist

**Example**

```bash
curl -X PUT https://api.getailicia.com/v1/characters/char_123/active \
  -H "Authorization: Bearer ${API_KEY}"
```

{{< callout context="note" title="Tip" >}}
List your characters first to confirm the `characterId` and ensure you have access.
{{< /callout >}}
