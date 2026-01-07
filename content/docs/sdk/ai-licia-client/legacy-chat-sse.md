---
title: Legacy chat SSE
type: docs
weight: 40
prev: /docs/sdk/ai-licia-client/eventsub
next: /docs/sdk/ai-licia-client/reference
description: Stream the legacy public chat SSE via ai_licia-client.
summary: streamPublicChatMessages example with a deprecation note.
---

{{< callout context="warning" title="Prefer EventSub" >}}
New integrations should use the unified EventSub stream via `streamEventSub`.
{{< /callout >}}

## Stream public chat messages

Connect to `/v1/events/chat/messages/stream` with optional role filtering.

```ts {filename="chat-stream.ts"}
const stream = client.streamPublicChatMessages({
  roles: ['AI', 'Streamer'],
  onOpen: () => console.log('Listening...'),
  onMessage: (msg) => console.log(`${msg.role}: ${msg.content}`),
  onError: (err) => console.error('Stream error', err),
  onClose: () => console.log('Stream closed')
});

// Later:
stream.close();
```
