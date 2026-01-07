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

Enrich ai_licia's context (max ~700 chars). Does not force an immediate response.

```ts {filename="send-event.ts"}
await client.sendEvent('Player health: 85%, Position: X:145 Y:230, Kills: 12');
await client.sendEvent('Current song: Never Gonna Give You Up');
```

With TTL (seconds):

```ts
await client.sendEvent('Lights: blue', 300);
```

## Trigger a reaction

Ask ai_licia to react now (max ~300 chars). Returns generation metadata.

```ts {filename="trigger.ts"}
const res = await client.triggerGeneration('Player just defeated the final boss!');
console.log(res);
```

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
