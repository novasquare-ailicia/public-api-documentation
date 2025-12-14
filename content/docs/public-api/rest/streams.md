---
title: Control Chat Presence
type: docs
weight: 76
prev: /docs/public-api/rest/characters
next: /docs/public-api/eventsub
description: Use POST/DELETE /v1/streams/{channelName} to ask ai_licia to join or leave your chat with proper requirements and sample responses.
---

Use these endpoints to ask ai_licia to join or leave one of the channels linked to your API key.

{{< callout context="tip" title="Requirements" >}}
- The channel must be owned by (or shared with) the API key owner.
- ai_licia must have an active subscription or available credits for the account to join.
{{< /callout >}}

## POST /v1/streams/{channelName}

Sends a join request and waits up to 10 seconds for the confirmation event.

**Request**
- Method/Path: `POST /v1/streams/{channelName}`
- Headers: `Authorization: Bearer <API_KEY>`
- Path parameters:
  - `channelName`: handle of a channel you control (e.g. your Twitch username)

**Responses**
- `200 OK` with a `JoinChannelResponse` payload indicating success, already-joined status, or timeout
- `401 Unauthorized` if the API key is invalid
- `403 Forbidden` if the channel is not associated with the key or no subscription/credits are available
- `404 Not Found` if the channel cannot be resolved for the streamer

**Example**

```bash
curl -X POST https://api.getailicia.com/v1/streams/mychannel \
  -H "Authorization: Bearer ${API_KEY}"
```

```json
{
  "success": true,
  "message": "AI successfully joined channel",
  "joinedAt": 1739982392000,
  "channelId": "ch_123"
}
```

If ai_licia is already connected:

```json
{
  "success": true,
  "message": "AI already in channel",
  "alreadyJoined": true
}
```

Timeout example:

```json
{
  "success": false,
  "message": "Join request sent but AI join confirmation timed out after 10 seconds",
  "timeout": true,
  "joinRequestId": "req_xyz"
}
```

## DELETE /v1/streams/{channelName}

Queues a leave request for the specified channel. The orchestrator processes it asynchronously, so the endpoint responds immediately.

**Request**
- Method/Path: `DELETE /v1/streams/{channelName}`
- Headers: `Authorization: Bearer <API_KEY>`
- Path parameters:
  - `channelName`: handle of a controlled channel

**Responses**
- `202 Accepted` once the leave request is enqueued
- `401 Unauthorized` if the API key is invalid
- `403 Forbidden` if the channel is not associated with the key
- `404 Not Found` if the channel cannot be resolved

**Example**

```bash
curl -X DELETE https://api.getailicia.com/v1/streams/mychannel \
  -H "Authorization: Bearer ${API_KEY}"
```
