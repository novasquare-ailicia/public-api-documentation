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
| `events` | Legacy local event notifications such as `ttsAmplitude` |
| `state` | Reserved generic app state capability |
| `integration.auth` | Access request flow is supported (available when user is signed in) |
| `avatar.protocol.v1` | Authenticated avatar subscription and event protocol |

See per-capability details:
- [Capability: Events](/docs/public-api/local-websocket/capabilities-events)
- [Capability: State](/docs/public-api/local-websocket/capabilities-state)
- [Capability: integration.auth](/docs/public-api/local-websocket/capabilities-integration-auth)
- [Avatar Protocol v1](/docs/public-api/local-websocket/avatar-protocol)

Avatar messages use the same JSON envelope. When an avatar subscription negotiates `binary-v1`, audio chunks are binary WebSocket messages instead; all handshake, subscription, lifecycle, prepared, completed, expression, action, and snapshot messages remain JSON.

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
    "version": "1.0.0",
    "reconnectCredential": "aig_opaque_credential"
  }
}
```

Fields:
- `clientType`: must be `integration`
- `clientId`: stable ID (slug or reverse-DNS)
- `displayName`: shown to the user (not verified)
- `version`: semantic version of your client
- `reconnectCredential`: optional opaque credential returned after earlier consent

### welcome (server -> client)
Server confirms support and capabilities.

```json
{
  "type": "welcome",
  "payload": {
    "capabilities": ["events", "state", "integration.auth", "avatar.protocol.v1"],
    "authorizedScopes": ["avatar.events.read"]
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
- `requestedScopes`: one or more supported scopes, including `rest.full` and `avatar.events.read`

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
User approved. The response contains only the credentials needed by the approved scopes.

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

An avatar-only grant does not expose the REST API key:

```json
{
  "type": "access_granted",
  "payload": {
    "requestId": "req_01HZXN5A",
    "scopes": ["avatar.events.read"],
    "reconnectCredential": "aig_opaque_credential"
  }
}
```

Persist the opaque credential in an application-owned file with operating-system appropriate permissions. Supply it in the next `hello` so approval survives restarts. A `clientId` alone is not authorization.

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
