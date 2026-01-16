---
title: Authentication
type: docs
weight: 10
prev: /docs/public-api
next: /docs/public-api/openapi
---

All public API requests use the channel API key:

```shell
Authorization: Bearer <API_KEY>
```

### Get your API key
1) Sign in to the ai_licia [Streamer Dashboard](https://dashboard.getailicia.com/).  
2) Go to **My account -> Settings** and generate/copy the API key.

![API key screenshot](/images/api_key.png)

### Local WebSocket consent
If you are building a local integration, the dashboard app can return the **channel API key** after the user approves your request. Use it the same way:

```shell
Authorization: Bearer <API_KEY>
```

See [Local WebSocket](/docs/public-api/local-websocket) for the consent flow.

Next:
- Send data: [REST Getting Started](/docs/public-api/rest/getting-started)
- Receive data: [EventSub Getting Started](/docs/public-api/eventsub/getting-started)
