---
title: Getting Started
type: docs
weight: 10
prev: /docs/public-api/local-websocket
next: /docs/public-api/local-websocket/messages
description: Connect to the local WebSocket, request scoped consent, and restore durable integration access.
summary: Step-by-step handshake for REST, Vision, and avatar integrations using the dashboard WebSocket.
---

## 1) Connect
On the same computer, Desktop listens on `ws://127.0.0.1:51743` by default (configurable via `INTEGRATION_WS_PORT`).

For another device on the same network, the user must enable Avatar LAN access in Desktop. Discover `_ailicia-avatar._tcp.local` with mDNS and connect to the advertised host and port using `ws://`. LAN clients can request only `avatar.events.read`; REST and Vision messages are rejected.

## 2) Send `hello`
Identify your app and declare the client type:

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

## 3) Read the `welcome`
The server responds with supported capabilities. `integration.auth` is present when the user is signed in and at least one authorization surface is available.

```json
{
  "type": "welcome",
  "payload": {
    "capabilities": ["events", "state", "integration.auth", "avatar.protocol.v1"],
    "authorizedScopes": []
  }
}
```

## 4) Request access
Send only the scopes you need. Use `rest.full` for the public REST API or `avatar.events.read` for avatar animation state and optional TTS audio.

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

The dashboard app will show a consent modal to the user. The `clientId` must match the earlier `hello`.

## 5) Handle the response
If `rest.full` is approved, you receive the channel API key:

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

For `avatar.events.read`, you receive an opaque reconnect credential instead of an API key. Persist it securely and include it as `reconnectCredential` in future `hello` messages. The next `welcome.authorizedScopes` confirms whether access was restored.

If the user denies, you receive:

```json
{
  "type": "access_denied",
  "payload": {
    "requestId": "req_01HZXN5A",
    "reason": "user_denied"
  }
}
```

## 6) Use the granted capability
Use the API key like any other API key:

```shell
Authorization: Bearer <API_KEY>
```

See the [REST endpoints](/docs/public-api/rest/endpoints) for REST access or [Avatar Protocol v1](/docs/public-api/local-websocket/avatar-protocol) for renderer subscriptions.

To revoke REST access, regenerate the API key in dashboard settings. Avatar grants can be revoked individually from the Desktop Integrations page.
