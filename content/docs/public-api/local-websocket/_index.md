---
title: Local WebSocket
type: docs
weight: 20
prev: /docs/public-api/authentication
next: /docs/public-api/local-websocket/getting-started
description: Use the authenticated Desktop WebSocket for API credentials, Vision setup, and renderer-neutral avatar events.
summary: Connect on the same computer or opt in to avatar-only LAN access, request scoped approval, and consume typed integration capabilities.
---

The Desktop app exposes one authenticated WebSocket for third-party integrations. REST and Vision access stay on the same computer. Users may explicitly enable avatar-only LAN access for renderers and physical avatars on the same network.

Key facts:
- Loopback URL: `ws://127.0.0.1:51743` (override with `INTEGRATION_WS_PORT`)
- LAN discovery: `_ailicia-avatar._tcp.local` when the user enables Avatar LAN access
- Transport: JSON control and event messages, with optional binary avatar audio chunks
- Authorization: explicit user consent with scope-specific revocation
- LAN boundary: only `avatar.events.read` and avatar protocol messages are accepted from non-loopback clients

Quick links:
{{< cards >}}
  {{< card link="/docs/public-api/local-websocket/getting-started" title="Getting Started" subtitle="Connect and request access" >}}
  {{< card link="/docs/public-api/local-websocket/messages" title="Message Reference" subtitle="Handshake and auth messages" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-events" title="Capability: Events" subtitle="TTS amplitude stream" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-state" title="Capability: State" subtitle="Reserved for future state snapshots" >}}
  {{< card link="/docs/public-api/local-websocket/capabilities-integration-auth" title="Capability: integration.auth" subtitle="Consent + API key flow" >}}
  {{< card link="/docs/public-api/local-websocket/avatar-protocol" title="Avatar Protocol v1" subtitle="Lifecycle, mouth animation, expressions, actions, and snapshots" >}}
  {{< card link="/docs/public-api/local-websocket/avatar-events" title="Avatar Event Reference" subtitle="Typed lifecycle, mouth, expression, action, and snapshot events" >}}
  {{< card link="/docs/public-api/local-websocket/avatar-audio" title="Avatar Audio" subtitle="PCM, Opus, Base64, and binary-v1 delivery" >}}
{{< /cards >}}
