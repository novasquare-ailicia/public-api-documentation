---
title: Message Reference
type: docs
weight: 20
prev: /docs/public-api/local-websocket/getting-started
next: /docs/public-api/local-websocket/capabilities-events
description: Message envelope and schemas for the local integration WebSocket.
summary: Typed message formats for handshake, access requests, and responses.
---

## Envelope
All messages use the same envelope:

```json
{
  "type": "hello",
  "payload": {}
}
```

## Capabilities
The `welcome` message returns the capabilities supported by the server:

| Capability | Description |
| --- | --- |
| `events` | Event notifications (future use) |
| `state` | App state snapshots (future use) |
| `integration.auth` | Access request flow is supported (available when user is signed in) |

See per-capability details:
- [Capability: Events](/docs/public-api/local-websocket/capabilities-events)
- [Capability: State](/docs/public-api/local-websocket/capabilities-state)
- [Capability: integration.auth](/docs/public-api/local-websocket/capabilities-integration-auth)

## Message Types

### hello (client -> server)
Identify your client.

```json
{
  "type": "hello",
  "payload": {
    "clientType": "integration",
    "clientId": "lumia",
    "displayName": "Lumia",
    "version": "1.0.0"
  }
}
```

Fields:
- `clientType`: must be `integration`
- `clientId`: stable ID (slug or reverse-DNS)
- `displayName`: shown to the user (not verified)
- `version`: semantic version of your client

### welcome (server -> client)
Server confirms support and capabilities.

```json
{
  "type": "welcome",
  "payload": {
    "capabilities": ["events", "state", "integration.auth"]
  }
}
```

### request_access (client -> server)
Request user approval for scopes.

```json
{
  "type": "request_access",
  "payload": {
    "clientId": "lumia",
    "displayName": "Lumia",
    "requestedScopes": ["rest.full"]
  }
}
```

Fields:
- `requestedScopes`: array of scopes; v1 supports `rest.full` only

### access_pending (server -> client, optional)
Indicates a consent prompt is waiting for user input.

```json
{
  "type": "access_pending",
  "payload": {
    "requestId": "req_01HZXN5A"
  }
}
```

### access_granted (server -> client)
User approved; channel API key is issued.

```json
{
  "type": "access_granted",
  "payload": {
    "requestId": "req_01HZXN5A",
    "apiKey": "sk_live_xxxxx",
    "scopes": ["rest.full"],
    "channelName": "mychannel"
  }
}
```

### access_denied (server -> client)
User denied the request.

```json
{
  "type": "access_denied",
  "payload": {
    "requestId": "req_01HZXN5A",
    "reason": "user_denied"
  }
}
```

### error (server -> client)
Typed error response.

```json
{
  "type": "error",
  "payload": {
    "code": "invalid_scope",
    "message": "Unsupported scope: rest.write"
  }
}
```
