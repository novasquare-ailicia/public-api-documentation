---
title: API reference
type: docs
weight: 50
prev: /docs/sdk/ai-licia-client/legacy-chat-sse
next: /docs/public-api
description: Quick reference of methods, exports, and environment variables for ai_licia-client.
summary: Methods, streams, and env vars at a glance.
---

## Exports

- `AiliciaClient`
- `GenerationMode`
- `GenerationOptions`
- Public types from `./interfaces` (EventSub types, payloads, stream options, and chat types)

## Methods

| Method | Purpose |
| --- | --- |
| `sendEvent(content, ttl?)` | Replace the single game context. |
| `sendIntegrationState(sourceId, content, ttl?)` | Replace one source in the integration block (upcoming SDK/backend release). |
| `clearIntegrationState(sourceId)` | Remove only that integration source (upcoming SDK/backend release). |
| `triggerGeneration(content, options?)` | Trigger an immediate AI response, optionally selecting `STANDARD` or `FAST`, TTS delivery, and a generation TTL. |
| `listCharacters()` | List characters available to the authenticated streamer. |
| `setActiveCharacter(characterId)` | Set the active character for the streamer. |
| `requestStreamJoin(channelName?)` | Ask ai_licia to join a channel. |
| `requestStreamLeave(channelName?)` | Ask ai_licia to leave a channel. |
| `streamEventSub(options?)` | Subscribe to the unified EventSub SSE stream. |
| `streamPublicChatMessages(options)` | Legacy public chat SSE (deprecated). |
| `AiliciaClient.getInstance(config)` | Get or create a singleton instance. |

## Environment variables

| Name | Purpose |
| --- | --- |
| `AI_LICIA_API_URL` | Override API base URL (default: `https://api.getailicia.com/v1`). |
| `AI_LICIA_API_KEY` | API key for authentication. |
| `AI_LICIA_CHANNEL` | Default channel name for requests. |
