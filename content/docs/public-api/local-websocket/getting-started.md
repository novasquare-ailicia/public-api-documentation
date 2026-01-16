---
title: Getting Started
type: docs
weight: 10
prev: /docs/public-api/local-websocket
next: /docs/public-api/local-websocket/messages
description: Connect to the local WebSocket, request user consent, and receive the channel API key.
summary: Step-by-step handshake for local integrations using the dashboard WebSocket.
---

## 1) Connect locally
The dashboard app listens on `ws://127.0.0.1:51743` by default (configurable via `INTEGRATION_WS_PORT`).

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
The server responds with supported capabilities. `integration.auth` is present when the user is signed in and the API key is available.

```json
{
  "type": "welcome",
  "payload": {
    "capabilities": ["events", "state", "integration.auth"]
  }
}
```

## 4) Request access
Send the scopes you need. v1 starts with a single scope: `rest.full`.

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

The dashboard app will show a consent modal to the user.

## 5) Handle the response
If approved, you receive the channel API key:

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

## 6) Call the REST API
Use the API key like any other API key:

```shell
Authorization: Bearer <API_KEY>
```

See the [REST endpoints](/docs/public-api/rest/endpoints) for available operations.

To revoke access, regenerate the API key in the dashboard settings.
