---
title: EventSub Overview
type: docs
weight: 10
prev: /docs/public-api/rest
next: /docs/public-api/eventsub/getting-started
description: Understand ai_licia’s EventSub SSE stream for chat, AI, moderation, channel, and system events.
---

EventSub is the single SSE stream for ai_licia’s public API. Use it to receive real-time events (chat, AI responses, moderation for AI messages, public events, join/leave, character changes). Use the REST API to send data in.

### What you get

| Capability | Details |
| --- | --- |
| Single stream | All events over one SSE; filter via `types` |
| Auth | Channel API key |
| Coverage | Chat + AI, channel/public events, system join/leave, character updates |
| TTS | Metadata only; audio stays on `/tts/{userId}/{browserKey}` |
| Writes | Send data via [REST API](/docs/public-api/rest) |

### Event keys at a glance

| Group | Keys |
| --- | --- |
| Chat | `chat.message`, `chat.ai_message`, `chat.first_message` |
| AI | `ai.thoughts`, `ai.tts.generated`, `ai.moderation` |
| Channel | `channel.event`, `api.event` |
| System | `system.join`, `system.left` |
| Character | `character.updated` |

Quick links:
{{< cards >}}
  {{< card link="/docs/public-api/eventsub/getting-started" title="Getting started" subtitle="Connect and filter events" >}}
  {{< card link="/docs/public-api/eventsub/events" title="Events reference" subtitle="Envelope + event payloads" >}}
  {{< card link="/docs/public-api/eventsub/legacy" title="Legacy chat SSE" subtitle="Deprecated chat-only stream" >}}
{{< /cards >}}
