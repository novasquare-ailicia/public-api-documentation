---
title: Send Context & Trigger Reactions
type: docs
weight: 60
prev: /docs/public-api/rest/getting-started
next: /docs/public-api/rest/characters
aliases:
  - /docs/public-api/rest/generations
description: Best-practice guide for POST /v1/events and POST /v1/events/generations, including use cases, payload tips, and limits.
summary: Scenario-driven reference for context ingestion and forced reactions, including rate limits and sample payloads.
---

Use two complementary endpoints to steer ai_licia in real time:

- `POST /v1/events` — stream contextual facts so she understands the situation.
- `POST /v1/events/generations` — demand an immediate response (rate-limited).

Both share the same envelope shape; choose the one that matches your use case.

## When to send events

| Situation | Suggested Endpoint | Notes & Sample Ideas |
| --- | --- | --- |
| Production cues or stream activity | `POST /v1/events` | Upcoming guests, sponsor reads, raid announcements, “BRB in 2 min”. |
| Game moments & quest progress | `POST /v1/events` or `POST /v1/events/generations` | Player deaths, scoring plays, engine failures, quest acceptance/completion, rare drops. Force a reaction when you want an immediate hype/condolence. |
| Setup & environment status | `POST /v1/events` | Lighting scenes, camera changes, mic mute/unmute, background music titles, smart-home statuses. |
| Creator-specific workflows | `POST /v1/events` | Cooking steps (“adding spices”, “sautéing onions”), art commissions (“inking outline complete”), IRL or maker updates. |
| Chat trigger that needs an instant line | `POST /v1/events/generations` | Emergency shouts, donation callouts, high-value channel point rewards. |
| Channel point reward that forces a quip | `POST /v1/events/generations` | “Roast me” redemptions, “suggest a song”, “pick the next ingredient” challenges. |
| Telemetry, HUD, and stat updates | `POST /v1/events` | Race lap times, split differentials, health/ammo, map zones, sensor readouts. Keeps ai_licia aligned with on-screen data without forcing an instant reply. |

{{< callout context="hint" title="Pick the lightest tool" >}}
`/v1/events` updates ai_licia's internal context and she decides whether to speak.  
`/v1/events/generations` **forces** a response right away. Use it sparingly to avoid interrupting ongoing flows.
{{< /callout >}}

### Scenario ideas for `/v1/events`

You can send rich variety of context snippets; below are starting points you can adapt per stream type:

- **Game events**: `"PLAYER_DOWN: squad wiped on floor 32, request motivational speech"`, `"Quest accepted: Rescue the lighthouse keeper"`, `"Engine failure on left turbine, emergency landing in progress"`.
- **Production cues / stream activity**: `"Starting Q&A segment, remind viewers to submit questions"`, `"Sponsor block in 5 minutes, mention Hydra Chairs"`, `"Switching to Just Chatting for 15 minutes"`.
- **Setup / environment**: `"Studio lights set to 'Night Mode'"`, `"Background track: Lofi Chill Mix Vol 2"`, `"Kitchen cam online, oven preheated to 375F"`.
- **Cooking / creative streams**: `"Recipe: Carbonara, currently boiling pasta"`, `"Art commission for @viewer42, blocking out colors"`, `"Maker stream: resin curing, 10 minutes remaining"`.
- **Telemetry / HUD data**: `"Lap 5/12 completed in 1:37.521, tire wear 45%"`, `"Health 23%, shield broken, ammo 12/90"`, `"Flight sim altitude 18,000ft, IAS 220 knots, crosswind 12 knots"`.

Blend multiple facts (within the ~700 char limit) to give ai_licia plenty of hooks.

## POST /v1/events (context ingestion) {#post-v1-events}

Feed ai_licia structured text so she can weave it into future replies.

**Typical payloads**
- Live game telemetry: `"Altitude 18 ft, heading 304, speed 5 knots"`
- Production notes: `"New sponsor goal unlocked, remind chat about the giveaway"`
- IRL trackers: `"Heart rate 120 bpm, say I'm sprinting"`

**Request**
- Method/Path: `POST /v1/events`
- Headers: `Authorization: Bearer <API_KEY>`, `Content-Type: application/json`
- Body:
  - `eventType` (string): categorize the event (e.g. `GAME_EVENT`, `IRL_EVENT`)
  - `data.channelName` (string): channel tied to your API key
  - `data.content` (string, max ~700 chars): the context snippet
  - `data.ttl` (optional integer): auto-expire after X seconds

**Responses**
- `200 OK` — ingested
- `401 Unauthorized` — missing/invalid key
- `422 Unprocessable Entity` — content too long

```bash
curl -X POST https://api.getailicia.com/v1/events \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "GAME_EVENT",
    "data": {
      "channelName": "mychannel",
      "content": "Squad reached Zone 3 with 2 medkits left; weather turning to storm."
    }
  }'
'
```

## POST /v1/events/generations (force a reply) {#post-v1-events-generations}

Request a specific reaction when you need ai_licia to speak now.

**Great for**
- Channel point rewards that demand a shout-out.
- Custom alerts (e.g. `"Tell chat we just hit 500 subs!"`).
- Emergency call-outs ("`Warn viewers we're about to wipe`").

**Request**
- Method/Path: `POST /v1/events/generations`
- Headers: `Authorization: Bearer <API_KEY>`, `Content-Type: application/json`
- Body:
  - `eventType` (string): use any label, e.g. `CHAT_TRIGGER`
  - `data.channelName` (string): channel tied to your API key
  - `data.content` (string, max ~300 chars): the prompt for ai_licia
  - `data.features` (object, optional): per-request feature flags
    - `vision` (boolean, default `false`)
    - `web_search` (boolean, default `false`)
    - `commands` (boolean, default `false`)
    - `memory` (boolean, default `false`)
    - `tts` (boolean, default `true`, still governed by existing TTS rules)

**Latency impact**
- Features run in parallel, so the total slowdown is driven by the slowest enabled feature.
- Approximate overhead when enabled:
  - `web_search`: +1.0s
  - `commands`: +1.2s
  - `memory`: +0.8s
  - `vision`: +7.0s
- `tts` does not delay posting the response, but audio delivery typically takes 500ms to 1s to generate and receive.

{{< callout context="info" title="Defaults" >}}
Omitting `data.features` (or any individual flag) uses the default values: all feature flags are `false` except `tts`, which defaults to `true` and still follows the existing TTS rules.
{{< /callout >}}

**Responses**
- `200 OK` — request accepted
- `401 Unauthorized` — missing/invalid key
- `422 Unprocessable Entity` — content too long
- `429 Too Many Requests` — exceeded 1 request per ~25-second window (per API key)

```bash
curl -X POST https://api.getailicia.com/v1/events/generations \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "CHAT_TRIGGER",
    "data": {
      "channelName": "mychannel",
      "content": "Tell chat the boss is enraged and we need focus now!",
      "features": {
        "vision": false,
        "web_search": true,
        "commands": false,
        "memory": true,
        "tts": true
      }
    }
  }'
'
```

{{< callout context="warning" title="Best practice" >}}
Let `/v1/events` drive most of the context; reserve `/v1/events/generations` for critical, infrequent prompts. The generation endpoint allows **one request roughly every 25 seconds per API key**, so queue/batch rewards when possible to avoid 429s.
{{< /callout >}}
