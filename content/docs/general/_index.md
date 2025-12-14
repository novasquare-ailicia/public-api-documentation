---
title: Introduction
type: docs
weight: 2
next: /docs/public-api
---

Welcome to the ai_licia platform documentation. This short primer orients you before diving into the Public API, REST endpoints, or EventSub stream.

## What you can build

- **Automations** – Feed real-time data (game state, alerts, custom triggers) so ai_licia can respond in chat and TTS.
- **Dashboards** – Monitor EventSub to visualize chat, AI activity, or moderation decisions in your control panels.
- **Extensions & bots** – Combine REST calls with EventSub streams to implement round-trip experiences, e.g., button presses that trigger AI callouts and reactions streamed back to your UI.

## Key surfaces

| Surface | Purpose | Quick links |
| --- | --- | --- |
| REST API | Send contextual events or request on-demand AI generations. | [REST overview](/docs/public-api/rest) |
| EventSub (SSE) | Receive chat, AI messages, moderation, channel events, and system updates in one stream. | [EventSub overview](/docs/public-api/eventsub) |
| SDKs | Official libraries to integrate faster (currently TypeScript/Node). | [SDKs](/docs/sdk) |

## Getting ready

1. Review [Authentication](/docs/public-api/authentication) to obtain or rotate your channel API key.
2. Use the [OpenAPI spec](/docs/public-api/openapi) or [ReDoc reference](/docs/public-api/api-reference) if you prefer working from schemas.
3. Pick your integration path:
   - **Write-first** (REST) if you only need to send context or triggers.
   - **Read-first** (EventSub) if you care about ai_licia’s responses, moderation, and channel system events.
   - **Both** for closed-loop automations.

Continue with the [Public API Overview](/docs/public-api) for deeper details on scopes, rate limits, and feature guides.
