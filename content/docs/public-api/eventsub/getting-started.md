---
title: Getting Started
type: docs
weight: 20
prev: /docs/public-api/eventsub
next: /docs/public-api/eventsub/events
description: Step-by-step instructions for authenticating and connecting to ai_licia’s EventSub SSE stream with filters and sample clients.
summary: Learn how to authenticate, filter `types`, and listen to EventSub SSE events via curl or browser EventSource.
---

## Prerequisites

- Channel API key (`Authorization: Bearer <API_KEY>`).
- SSE-capable client (for example, `EventSource` in browsers).

## Steps
1) Get your API key (see [Authentication](/docs/public-api/authentication)).  
2) Pick `types` to filter (optional).  
3) Connect to the SSE endpoint.  
4) Handle events by `event` name (matches `type`).

## Endpoint

```shell
GET /v1/eventsub/stream
Authorization: Bearer <API_KEY>
```

**Query parameters**

| Name | Description | Example |
| --- | --- | --- |
| `types` | Comma-separated event keys to include | `types=chat.message,ai.tts.generated` |
| `channelId` | Target a specific channel (if the key has multiple) | `channelId=ch_123` |
## Quick start (curl)

```bash
curl -N "https://api.getailicia.com/v1/eventsub/stream?types=chat.message,ai.tts.generated" \
  -H "Authorization: Bearer ${API_KEY}"
```

## Client example (browser)

```javascript
const es = new EventSource(
  'https://api.getailicia.com/v1/eventsub/stream?types=chat.message,ai.tts.generated',
  { withCredentials: false }
);

es.addEventListener('chat.message', (evt) => {
  const envelope = JSON.parse(evt.data);
  console.log('New chat message', envelope.payload);
});

es.addEventListener('system.ping', () => {
  // keep-alive, no action required
});
```

## Notes

{{< callout context="info" title="Framing" >}}
SSE `event` names match the `type` field in the envelope (e.g. `chat.message`). A `system.ping` is emitted periodically to keep the connection alive.
{{< /callout >}}
