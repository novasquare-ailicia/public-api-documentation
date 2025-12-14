---
title: ai_licia-client (npm)
type: docs
weight: 80
prev: /docs/sdk
next: /docs/public-api
description: Reference for the official ai_licia TypeScript/JavaScript client to send REST events, trigger generations, and stream chat SSE.
---

TypeScript/JavaScript client for the ai_licia public API: send contextual events, trigger reactions, and consume the public chat SSE.

[View on npm](https://www.npmjs.com/package/ai_licia-client)

## Install

```bash
npm install ai_licia-client
```

## Configure

Env vars (preferred):
```bash
AI_LICIA_API_URL=https://api.getailicia.com/v1
AI_LICIA_API_KEY=your-api-key
AI_LICIA_CHANNEL=your-channel-name
```

Or pass values to the constructor.

## Initialize

```ts {filename="init.ts"}
import { AiliciaClient } from 'ai_licia-client';

const client = new AiliciaClient(
  process.env.AI_LICIA_API_KEY,      // optional, uses env if absent
  process.env.AI_LICIA_CHANNEL,      // optional, uses env if absent
  process.env.AI_LICIA_API_URL       // optional, defaults to https://api.getailicia.com/v1
);
```

## Send contextual events

Enrich ai_licia’s context (max ~700 chars). Does not force an immediate response.

```ts {filename="send-event.ts"}
await client.sendEvent('Player health: 85%, Position: X:145 Y:230, Kills: 12');
await client.sendEvent('Current song: Never Gonna Give You Up');
```

With TTL (seconds):
```ts
await client.sendEvent('Lights: purple', 300);
```

## Trigger a reaction

Ask ai_licia to react now (max ~300 chars). Returns generation metadata.

```ts {filename="trigger.ts"}
const res = await client.triggerGeneration('Player just defeated the final boss!');
console.log(res);
```

## Stream public chat messages (SSE)

Connect to `/v1/events/chat/messages/stream` with optional role filtering.

```ts {filename="stream.ts"}
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

## Errors

API errors throw with messages for length limits, auth, and rate limits (1 forced generation ~every 25s per API key).

```ts
try {
  await client.sendEvent('Player entered the final dungeon');
} catch (err: any) {
  console.error('Failed to send event:', err.message);
}
```

## Quick reference

| Item | Value |
| --- | --- |
| Package | `ai_licia-client` |
| Export | `AiliciaClient` |
| Methods | `sendEvent(content, ttl?)`, `triggerGeneration(content)`, `streamPublicChatMessages(options)` |
| Env vars | `AI_LICIA_API_URL`, `AI_LICIA_API_KEY`, `AI_LICIA_CHANNEL` |

Related:
- REST API: [Getting started](/docs/public-api/rest/getting-started)
- EventSub: [Getting started](/docs/public-api/eventsub/getting-started)
