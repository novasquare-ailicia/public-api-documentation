---
title: REST usage
type: docs
weight: 20
prev: /docs/sdk/ai-licia-client/getting-started
next: /docs/sdk/ai-licia-client/eventsub
description: Send contextual events and trigger generations via REST.
summary: sendEvent and triggerGeneration examples with optional channel helpers.
---

## Send contextual events

Enrich ai_licia's context (max 1,000 chars). Does not force an immediate response.

```ts {filename="send-event.ts"}
await client.sendEvent('Player health: 85%, Position: X:145 Y:230, Kills: 12');
await client.sendEvent('Current song: Never Gonna Give You Up');
```

With TTL (seconds):

```ts
await client.sendEvent('Lights: blue', 300);
```

## Trigger a reaction

Ask ai_licia to react now (max 300 chars). Returns generation metadata.

```ts {filename="trigger.ts"}
const res = await client.triggerGeneration('Player just defeated the final boss!');
console.log(res);
```

For a brief, time-sensitive reaction, select fast model execution while keeping TTS independently controllable:

```ts {filename="fast-trigger.ts"}
import { GenerationMode } from 'ai_licia-client';

await client.triggerGeneration(
  'Tyre puncture. Briefly tell the driver to pit safely.',
  {
    mode: GenerationMode.FAST,
    tts: true,
    ttl: 8,
  },
);
```

Fast mode retains the active persona, configured language, and compact stream environment. It excludes history, memory, knowledge-base content, goals, mission, vision, web search, and commands, and makes one model attempt without fallback. It does not change the accepted content you provide. TTS remains independently controllable.

`STANDARD` and `FAST` use independent per-key rate windows: roughly 25 seconds for `STANDARD`, and 1 second for `FAST`. `ttl` is optional and measured in seconds. It prevents a stale request from being processed after its deadline.

Use the default `STANDARD` mode when minimum latency is not required or the response needs any excluded context or capability.

## Channel helpers

```ts {filename="channel.ts"}
const characters = await client.listCharacters();
await client.setActiveCharacter(characters[0].id);

const join = await client.requestStreamJoin();
if (!join.success) {
  console.warn('Join failed', join.message);
}

await client.requestStreamLeave('backupChannel');
```
