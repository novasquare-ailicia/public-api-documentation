---
title: EventSub stream
type: docs
weight: 30
prev: /docs/sdk/ai-licia-client/rest
next: /docs/sdk/ai-licia-client/legacy-chat-sse
description: Stream EventSub SSE with filters and handlers via ai_licia-client.
summary: streamEventSub example with filters, handlers, and reconnects.
---

Use `streamEventSub` to connect to the unified SSE stream at `/v1/eventsub/stream`.

## Usage example

```ts {filename="eventsub.ts"}
const stream = client.streamEventSub({
  types: ['chat.message', 'ai.tts.generated'],
  autoReconnect: true,
  onOpen: () => console.log('EventSub connected'),
  onError: (err) => console.error('EventSub error', err),
  handlers: {
    'chat.message': (event) => {
      console.log(`[${event.payload.platform}] ${event.payload.username}: ${event.payload.message}`);
    }
  }
});

stream
  .on('ai.tts.generated', (event) => {
    console.log(`New TTS clip at ${event.payload.ttsStreamUrl}`);
  })
  .onAny((event) => console.log(`Received ${event.type}`));

// Stop streaming
stream.close();
```

## Filters and options

| Option | Description |
| --- | --- |
| `types` | Limit to specific event keys (comma list on the wire). |
| `channelId` | Target a specific channel when the API key controls multiple. |
| `cursor` | Opaque resume token for future replay support. |
| `handlers` | Map of per-event handlers, invoked on message receipt. |
| `autoReconnect` | Enable automatic reconnects (default: true). |
| `onReconnectAttempt` | Called with `{ attempt, delayMs }` before each retry. |
| `onConnectionStateChange` | Notifies `connecting` and `reconnecting` states. |

Event names match the `type` field in the EventSub envelope. For full payload schemas, see [EventSub docs](/docs/public-api/eventsub).
