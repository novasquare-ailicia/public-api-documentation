---
title: "Capability: State"
type: docs
weight: 40
prev: /docs/public-api/local-websocket/capabilities-events
next: /docs/public-api/local-websocket/capabilities-integration-auth
description: State capability for the local WebSocket.
summary: Reserved capability; no state messages are emitted yet.
---

The `state` capability is reserved for future local state snapshots (for example, connection status or app state).

Current behavior:
- The server does not emit any `state` messages yet.
- Treat this capability as a placeholder and do not depend on it.

Check the `welcome` capabilities list and only use the features you actually see documented and implemented.
