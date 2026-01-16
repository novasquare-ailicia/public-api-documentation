---
title: Local WebSocket
type: docs
weight: 20
prev: /docs/public-api/authentication
next: /docs/public-api/local-websocket/getting-started
description: Use the local dashboard WebSocket to request user consent and receive the channel API key.
summary: Connect locally, request approval, and receive the channel API key without manual copy/paste.
---

The desktop dashboard app exposes a **local-only WebSocket** for third-party integrations (for example, Lumia). It lets your app request user approval and receive the **channel API key** that works with the REST API.

Key facts:
- URL: `ws://127.0.0.1:51743` (override with `INTEGRATION_WS_PORT`)
- Transport: JSON messages with `{type, payload}`
- Authorization: explicit user consent; revoke by regenerating the API key
- Local only: the server binds to `127.0.0.1`

Quick links:
{{< cards >}}
  {{< card link="/docs/public-api/local-websocket/getting-started" title="Getting Started" subtitle="Connect and request access" >}}
  {{< card link="/docs/public-api/local-websocket/messages" title="Message Reference" subtitle="Handshake and auth messages" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-events" title="Capability: Events" subtitle="TTS amplitude stream" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-state" title="Capability: State" subtitle="Reserved for future state snapshots" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-integration-auth" title="Capability: integration.auth" subtitle="Consent + API key flow" >}}
{{< /cards >}}
