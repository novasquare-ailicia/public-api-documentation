---
title: "Capability: integration.auth"
type: docs
weight: 50
prev: /docs/public-api/local-websocket/capabilities-state
next: /docs/public-api/rest
description: User consent flow for local integrations to access the channel API key.
summary: Request access, wait for user approval, then use the returned API key.
---

The `integration.auth` capability enables a consent flow that returns the **channel API key** after the user approves your request.

## When is it available?
`integration.auth` appears in the `welcome` capabilities when:
- The user is signed in, and
- The dashboard app has loaded the channel API key.

If it is missing, your client should fall back or retry later.

## Request flow
1) Send `hello` with `clientType: "integration"`.
2) Send `request_access` with `clientId`, `displayName`, and `requestedScopes`.
3) The user sees a consent modal in the dashboard app.
4) You receive either `access_granted` or `access_denied`.

See the full message schemas in [Message Reference](/docs/public-api/local-websocket/messages).

## Scopes
v1 supports only:
- `rest.full`

Any other scope triggers an error.

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
