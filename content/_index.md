---
title: ai_licia API & SDKs
toc: false
description: Official ai_licia REST, EventSub, and SDK documentation for building AI co-host experiences.
summary: Overview of all ai_licia API surfaces, including REST ingestion, EventSub streaming, and SDKs to build AI co-hosts.
---

# Build with ai_licia’s public API

Server-side reactions, real-time events, and ready-made SDKs to power AI co-host experiences.

## Jump in

{{< cards >}}
  {{< card link="docs/public-api/authentication" title="Get your API key" subtitle="One header for REST + EventSub" >}}
  {{< card link="docs/public-api/rest/openapi" title="OpenAPI spec" subtitle="Download or fetch spec.yaml" >}}
  {{< card link="docs/public-api/rest/api-reference" title="API Reference" subtitle="Interactive ReDoc view of the spec" >}}
  {{< card link="docs/public-api/rest" title="REST API" subtitle="Send contextual events; trigger reactions" >}}
  {{< card link="docs/public-api/eventsub" title="EventSub SSE" subtitle="Stream chat, AI, channel, and system events" >}}
  {{< card link="docs/sdk" title="SDKs & Libraries" subtitle="Use the npm client or other tooling" >}}
{{< /cards >}}

## What you can build

{{< cards >}}
  {{< card link="docs/public-api/rest/getting-started" title="Context ingestion" subtitle="Push game/telemetry/state (700 chars)" >}}
  {{< card link="docs/public-api/rest/endpoints" title="Instant reactions" subtitle="Trigger ai_licia responses (300 chars)" >}}
  {{< card link="docs/public-api/eventsub/events" title="Unified events" subtitle="Chat, AI messages, moderation, channel events, system events" >}}
  {{< card link="docs/sdk/ai-licia-client" title="npm: ai_licia-client" subtitle="Typed JS/TS client for REST and chat SSE" >}}
{{< /cards >}}

## Start fast

1) Grab your key from the [Dashboard](https://dashboard.getailicia.com/).  
2) Send your first event: [REST Getting Started](docs/public-api/rest/getting-started).  
3) Listen to outputs: [EventSub Getting Started](docs/public-api/eventsub/getting-started).
