---
title: Getting started
type: docs
weight: 10
prev: /docs/sdk/ai-licia-client
next: /docs/sdk/ai-licia-client/rest
description: Install and initialize the ai_licia-client npm package.
summary: npm install, environment variables, and the AiliciaClient constructor.
---

## Install

```bash
npm install ai_licia-client
```

## Configure

Set environment variables (recommended):

```bash
AI_LICIA_API_URL=https://api.getailicia.com/v1
AI_LICIA_API_KEY=your-api-key
AI_LICIA_CHANNEL=your-channel-name
```

If you keep a `.env` file in your project root, the client will try to load it automatically.

## Initialize

```ts {filename="init.ts"}
import { AiliciaClient } from 'ai_licia-client';

const client = new AiliciaClient(
  process.env.AI_LICIA_API_KEY,      // optional, uses env if absent
  process.env.AI_LICIA_CHANNEL,      // optional, uses env if absent
  process.env.AI_LICIA_API_URL       // optional, defaults to https://api.getailicia.com/v1
);
```

## Next steps

- Send REST events with `sendEvent` and `triggerGeneration`.
- Subscribe to EventSub with `streamEventSub`.
