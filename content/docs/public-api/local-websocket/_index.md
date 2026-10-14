---
title: Local WebSocket
type: docs
weight: 20
prev: /docs/public-api/authentication
next: /docs/public-api/local-websocket/getting-started
description: Use the authenticated local dashboard WebSocket for API credentials, Vision setup, and renderer-neutral avatar events.
summary: Connect locally, request scoped approval, and consume typed integration capabilities.
---

The desktop dashboard app exposes a **local-only WebSocket** for third-party integrations. It provides explicit, scoped consent for REST access, Vision setup, and renderer-neutral avatar events.

Key facts:
- URL: `ws://127.0.0.1:51743` (override with `INTEGRATION_WS_PORT`)
- Transport: JSON messages with `{type, payload}`
- Authorization: explicit user consent with scope-specific revocation
- Local only: the server binds to `127.0.0.1`

Quick links:
{{< cards >}}
  {{< card link="/docs/public-api/local-websocket/getting-started" title="Getting Started" subtitle="Connect and request access" >}}
  {{< card link="/docs/public-api/local-websocket/messages" title="Message Reference" subtitle="Handshake and auth messages" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-events" title="Capability: Events" subtitle="TTS amplitude stream" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-state" title="Capability: State" subtitle="Reserved for future state snapshots" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-integration-auth" title="Capability: integration.auth" subtitle="Consent + API key flow" >}}
  {{< card link="/docs/public-api/local-websocket/avatar-protocol" title="Avatar Protocol v1" subtitle="Lifecycle, mouth animation, expressions, actions, and snapshots" >}}
{{< /cards >}}
