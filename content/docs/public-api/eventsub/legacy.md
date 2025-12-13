---
title: Legacy Chat SSE
type: docs
weight: 40
prev: /docs/public-api/eventsub/events
next: /docs/public-api/rest
---

> Legacy endpoint kept for backward compatibility. Prefer the unified EventSub stream for new integrations.

## Endpoint

```shell
GET /v1/events/chat/messages/stream
Authorization: Bearer <API_KEY>
```

## Access requirements
- Channel API key required.
- Streamer must have an active Flex, Unlimited, or Ultimate subscription (or credit).

## Behavior
- Streams only chat messages (viewer/streamer) and AI chat messages for the primary channel of the API key.
- Optional `roles` query filter (repeatable): `Mod`, `VIP`, `AI`, `Viewer`, `Streamer`.
- Sends periodic `:ping` comments every 30s to keep the connection alive.

## Payload
`event: message` with data:

```json
{
  "username": "viewer42",
  "content": "Let's go!",
  "role": "Viewer",        // one of Mod | VIP | AI | Viewer | Streamer
  "isSub": true,
  "sentDateTime": "2025-02-18T21:15:22Z"
}
```

## Notes
- Channel is inferred from the API key (primary channel).
- Use EventSub to receive richer event types (chat, AI responses, moderation, channel events, etc.) on a single SSE connection.

{{< callout context="warning" title="Prefer EventSub" >}}
New integrations should use [EventSub](/docs/public-api/eventsub) instead of this legacy chat stream.
{{< /callout >}}
