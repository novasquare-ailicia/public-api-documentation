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

Blend multiple facts (within the 1,000-character limit) to give ai_licia plenty of hooks.

## POST /v1/events (context ingestion) {#post-v1-events}

Feed ai_licia a complete current snapshot. Sending context does not force an immediate response.

> **Upcoming:** `INTEGRATION_STATE` requires the new backend rollout. Existing `GAME_EVENT` requests remain compatible. The new SDK helpers below require the next client release.

Context is assembled into two independent blocks:

| `eventType` | Replacement behaviour |
| --- | --- |
| `GAME_EVENT` | One game snapshot per channel. Every call replaces the entire game block. Use one active game integration. |
| `INTEGRATION_STATE` | One snapshot per `data.sourceId`. A call replaces only that source; active sources are combined into a separate integration block. |

For example, an OBS update leaves Lumia, Spotify and the current game intact. A new game snapshot leaves every integration entry intact. Integration payloads are complete replacements, not patches or event histories.

**Request**

- Method/Path: `POST /v1/events`
- Headers: `Authorization: Bearer <API_KEY>`, `Content-Type: application/json`
- Body:
  - `eventType`: `GAME_EVENT` or `INTEGRATION_STATE`. `IRL_EVENT` is not supported.
  - `data.channelName`: channel tied to your API key.
  - `data.content`: the complete snapshot, max 1,000 characters.
  - `data.sourceId`: required only for `INTEGRATION_STATE`. Use a stable identifier such as `obs`, `lumia` or `spotify`. It must match `[a-z0-9][a-z0-9._-]{0,63}`. IDs are case-sensitive and scoped to the authenticated channel; using the same ID intentionally replaces that entry. IDs identify context entries and do not grant separate access permissions.
  - `data.ttl`: validity in seconds. For integrations, an integer from 1 to 86,400, default 60 if omitted or null. Every update refreshes only its own expiry. For the existing game type, omitted or null TTL retains its existing non-expiring behaviour.

Refresh before expiry while connected. To disconnect, send the same type and source ID with empty `content`; this removes only that integration entry. Empty game content clears only the game block. Whitespace-only integration content also clears the entry.

The integration block retains up to **sixteen active sources**. An update to an existing source does not use another slot. Expired sources are removed first; adding a seventeenth active source removes the least recently updated entry. This bounds the total context supplied to the model. Entries belong to the active AI session and survive worker relocation; integrations should resend their snapshot after a new session starts. Context ingestion is not a durable memory API.

**Responses**

- `200 OK`: ingested; no immediate reaction is requested.
- `401 Unauthorized`: missing/invalid key or channel mismatch.
- `422 Unprocessable Entity`: content too long, missing/invalid integration source ID, integration TTL outside the accepted range, or `sourceId` supplied for `GAME_EVENT`.

Game snapshot:

```bash
curl -X POST https://api.getailicia.com/v1/events \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "GAME_EVENT",
    "data": {
      "channelName": "mychannel",
      "content": "Day 12, raining. Player at the farm with 80 energy.",
      "ttl": 60
    }
  }'
```

Independent integration snapshot:

```json
{
  "eventType": "INTEGRATION_STATE",
  "data": {
    "channelName": "mychannel",
    "sourceId": "obs",
    "content": "Streaming. Current scene: Gameplay.",
    "ttl": 60
  }
}
```

Disconnect OBS without clearing the game or other integrations:

```json
{
  "eventType": "INTEGRATION_STATE",
  "data": {
    "channelName": "mychannel",
    "sourceId": "obs",
    "content": ""
  }
}
```

Keep `GAME_EVENT` for games. Migrate existing non-game callers to `INTEGRATION_STATE` with a stable source ID. Clients that continue sending `GAME_EVENT` still replace the single game block.

## POST /v1/events/generations (force a reply) {#post-v1-events-generations}

Request a specific reaction when you need ai_licia to speak now. This endpoint continues to accept `GAME_EVENT` only; `INTEGRATION_STATE` belongs on `/v1/events` and is rejected here with `422`. Generations do not replace either context block.

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
  - `data.content` (string, max 300 chars): the prompt for ai_licia
  - `data.options` (object, optional): execution and delivery options
    - `mode` (`STANDARD` or `FAST`, default `STANDARD`)
    - `tts` (boolean, default `true`, still governed by existing TTS rules)
    - `ttl` (optional positive integer, seconds): discard the request if processing has not started before it becomes stale

{{< callout context="info" title="Defaults" >}}
Omitting `data.options` uses `STANDARD` model execution with TTS enabled. You can control TTS independently in either mode.
{{< /callout >}}

### Standard and fast modes

`STANDARD` is the default. Use it for normal reactions, summaries, or requests that need conversation history, memory, knowledge-base content, goals, mission, vision, web search, or commands.

`FAST` selects a dedicated latency-optimized path for brief, time-sensitive reactions. It keeps the active character and persona, configured language, compact streamer identity and description, current stream category, and current stream title. It deliberately excludes conversation history, memory, knowledge-base content, goals, mission, vision, web search, and commands.

Fast mode does not rewrite, shorten, or otherwise change the accepted `data.content`. Send the complete instruction and factual grounding the message needs. If a particular sentence style is useful, include it as a hint in the content itself.

Fast mode uses one model attempt with no fallback so stale messages are not delayed by recovery work. This trades some quality and resilience for lower latency. It does not guarantee a fixed end-to-end response or audio delivery time. TTS remains independently controlled by `data.options.tts`.

Rate limits are independent by mode for each API key: `STANDARD` allows one accepted request roughly every 25 seconds, while `FAST` allows one accepted request per second. A request in one mode does not consume the other mode's window. These limits control API admission, not guaranteed speech cadence. Overlapping generations may still be serialized by downstream delivery.

**Responses**
- `200 OK` — request accepted
- `401 Unauthorized` — missing/invalid key
- `422 Unprocessable Entity` — content too long
- `429 Too Many Requests` — exceeded the selected mode's per-key window (`STANDARD`: ~25 seconds, `FAST`: 1 second)

```bash
curl -X POST https://api.getailicia.com/v1/events/generations \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "CHAT_TRIGGER",
    "data": {
      "channelName": "mychannel",
      "content": "Tell chat the boss is enraged and we need focus now!",
      "options": {
        "mode": "FAST",
        "tts": true,
        "ttl": 8
      }
    }
  }'
'
```

{{< callout context="warning" title="Best practice" >}}
Let `/v1/events` drive durable context. Use `STANDARD` generations for normal reactions and summaries, and reserve `FAST` for brief facts that become stale quickly. Respect the independent per-key windows of **roughly 25 seconds for `STANDARD`** and **1 second for `FAST`**. Set a short `ttl` on live alerts so delayed advice is discarded rather than spoken late.
{{< /callout >}}
