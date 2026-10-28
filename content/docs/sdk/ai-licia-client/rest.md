---
title: REST usage
type: docs
weight: 20
prev: /docs/sdk/ai-licia-client/getting-started
next: /docs/sdk/ai-licia-client/eventsub
description: Send contextual events and trigger generations via REST.
summary: sendEvent and triggerGeneration examples with optional channel helpers.
---

## Send game state

`sendEvent` replaces the single current game snapshot (max 1,000 characters). Use one active game integration. It does not force an immediate response.

```ts
await client.sendEvent('Day 12, raining. Player at the farm.', 60);
```

## Send integration state

> **Upcoming:** these helpers require the next SDK release and the new backend rollout.

Each stable source replaces only its own entry. Entries are combined in a separate integration block; they do not overwrite game state.

```ts
await client.sendIntegrationState('obs', 'Streaming. Scene: Gameplay.', 60);
await client.sendIntegrationState('lumia', 'Lights: purple.', 120);
await client.sendIntegrationState('spotify', 'Current track: Example song.');
await client.clearIntegrationState('obs');
```

Source IDs match `[a-z0-9][a-z0-9._-]{0,63}`. Each payload is at most 1,000 characters. Integration TTL defaults to 60 seconds and accepts integers from 1 to 86,400. Refresh while connected and clear on disconnect. Up to eight sources are retained; the least recently updated is removed if a ninth active source is added. See [context replacement and expiry](/docs/public-api/rest/events#post-v1-events).

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
