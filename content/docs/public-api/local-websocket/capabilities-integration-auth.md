---
title: "Capability: integration.auth"
type: docs
weight: 50
prev: /docs/public-api/local-websocket/capabilities-state
next: /docs/public-api/rest
description: User consent flow for local integrations to obtain narrowly scoped access.
summary: Request REST, Vision, or avatar access and restore durable grants securely.
---

The `integration.auth` capability enables a consent flow for narrowly scoped local integration access. Credentials are returned only when required by the approved scope.

## When is it available?
`integration.auth` appears in the `welcome` capabilities when the user is signed in and at least one integration authorization surface is available.

If it is missing, your client should fall back or retry later.

## Request flow
1) Send `hello` with `clientType: "integration"`.
2) Send `request_access` with `clientId`, `displayName`, and `requestedScopes`.
3) The user sees a consent modal in the dashboard app.
4) You receive either `access_granted` or `access_denied`.

See the full message schemas in [Message Reference](/docs/public-api/local-websocket/messages).

## Scopes

| Scope | Purpose | Returned credential |
| --- | --- | --- |
| `rest.full` | Use the account REST API | Channel API key |
| `avatar.events.read` | Subscribe to local avatar animation events | Opaque reconnect credential |
| `vision.rtmp.configure` | Configure the legacy Vision publishing output | Vision RTMP credential |
| `vision.installation.register` | Link a managed Vision installation | Installation authorization |

Request only the scopes your integration needs.

## Responses
Success:

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

Avatar-only success:

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

The reconnect credential is bound to the current ai_licia account, Desktop installation, and integration `clientId`. Desktop persists only a credential hash. The integration must store its opaque credential and supply it in a later `hello`. Users can revoke avatar grants from the Desktop Integrations page.

Denial:

```json
{
  "type": "access_denied",
  "payload": {
    "requestId": "req_01HZXN5A",
    "reason": "user_denied"
  }
}
```

Possible `reason` values:
- `user_denied`
- `timeout`
- `not_ready`

## Error codes
Errors are sent with `type: "error"` and a typed `code`:
- `missing_hello`
- `invalid_client_type`
- `invalid_client_id`
- `invalid_scope`
- `not_ready`
- `request_pending`
- `rate_limited`
- `ui_unavailable`

## Rate limits and timeouts
- One pending request per `clientId`.
- Requests within a short cooldown window are rejected (`rate_limited`).
- Pending requests expire automatically after a short timeout.

## Revocation
Integrations receive the channel API key. The user can revoke access by regenerating the API key in the dashboard settings.
