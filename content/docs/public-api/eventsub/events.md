---
title: Events Reference
type: docs
weight: 30
prev: /docs/public-api/eventsub/getting-started
next: /docs/public-api/eventsub/legacy
---

## Envelope

All events share the same envelope. Fields marked optional may be omitted if not applicable.

```json
{
  "id": "uuid",
  "type": "chat.message",
  "version": "2025-02-01",
  "timestamp": "2025-02-18T21:15:22Z",
  "channel": {
    "id": "ch_123",
    "name": "mychannel",
    "platform": "TWITCH"
  },
  "stream": {
    "id": "st_456"
  },
  "payload": { /* event-specific body */ },
  "meta": { /* event-specific metadata */ }
}
```

SSE frames use `event: <type>` and `data: <envelope JSON>`. Pings use `event: system.ping`.

Quick jump:
- [`chat.message`](#chatmessage)
- [`chat.ai_message`](#chataimessage)
- [`chat.first_message`](#chatfirst_message)
- [`ai.thoughts`](#aithoughts)
- [`ai.tts.generated`](#aittsgenerated)
- [`channel.event`](#channelevent)
- [`ai.moderation`](#aimoderation)
- [`api.event`](#apievent)
- [`system.join`](#systemjoin)
- [`system.left`](#systemleft)
- [`character.updated`](#characterupdated)
- [`system.ping`](#systemping)

## Event catalog

| Key | Description | Payload highlights |
| --- | --- | --- |
| `chat.message` | Viewer or streamer chat message. | id, username, message, language, platform, isSubscriber, isVip, isModerator, sentDateTime |
| `chat.ai_message` | AI-authored chat message. | id, username, message, target, targetRole, shouldGenerateTtsMessage, characterId, tone, sentDateTime |
| `chat.first_message` | Memory fragment surfaced when the AI joins; delivers the resolved memory (and optional greeting) for the chatter the AI just saw. Not a first-message indicator. | memory {id, name, facts, updated}, greeting?, username, sentDateTime |
| `ai.thoughts` | Humanized AI reasoning (raw thoughts are not emitted). | reasoning, shouldAnswer, compiledFromMultiple, sentDateTime |
| `ai.tts.generated` | TTS job metadata for an AI message. | messageId, textToSpeech, username, audioFormat, generationTimeMs, audioSizeBytes, ttsStreamUrl |
| `channel.event` | Platform events (subs, cheers, raids, follows, etc.). | eventType, username, gifter, count, value, raidViewers, message, sentDateTime |
| `ai.moderation` | AI-authored message moderation result. | messageId, username, originalMessage, isAppropriate, categoriesScore, moderationEnabled, sentDateTime |
| `api.event` | Custom public event sent via the API. | eventType, content, shouldGenerate, ttl, expiresAt |
| `system.join` | AI joined the channel. | channelId, channelName, sentDateTime |
| `system.left` | AI left the channel. | channelId, channelName, sentDateTime |
| `character.updated` | Active character changed. | characterId, displayName, sentDateTime |
| `system.ping` | Keep-alive ping. | pingId |

## Schemas
Examples below are shown as full envelopes: `type` matches the SSE event name, and `payload` carries the event-specific body.

{{< callout context="info" title="Event names" >}}
SSE `event` equals the `type` field in the envelope (e.g. `chat.message`).
{{< /callout >}}

### chat.message
- `id`: message identifier  
- `username`: chatter username  
- `message`: text content  
- `language`: ISO language code  
- `platform`: source platform (e.g. TWITCH, YOUTUBE)  
- `isSubscriber`, `isVip`, `isModerator`: booleans  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_chat_msg_1",
  "type": "chat.message",
  "timestamp": "2025-02-18T21:15:22Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "id": "msg_1",
    "username": "viewer42",
    "message": "Let's go!",
    "language": "en",
    "platform": "TWITCH",
    "isSubscriber": true,
    "isVip": false,
    "isModerator": false,
    "sentDateTime": "2025-02-18T21:15:22Z"
  }
}
```

### chat.ai_message
- `id`: message identifier  
- `username`: AI display name  
- `message`: text content  
- `target`: target username or `chat`  
- `targetRole`: viewer role (e.g. MOD, VIP, VIEWER)  
- `shouldGenerateTtsMessage`: whether TTS should be generated  
- `characterId`: active character id (nullable)  
- `tone`: optional tone tag  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_ai_msg_1",
  "type": "chat.ai_message",
  "timestamp": "2025-02-18T21:15:30Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "id": "ai_msg_1",
    "username": "ailicia",
    "message": "I'll cover that in a moment!",
    "target": "chat",
    "targetRole": "VIEWER",
    "shouldGenerateTtsMessage": true,
    "characterId": "char_123",
    "tone": "friendly",
    "sentDateTime": "2025-02-18T21:15:30Z"
  },
  "meta": { "shouldGenerateTts": true }
}
```

### chat.first_message
- `memory`: primary payload; memory block `{ id, name, facts, updated }` when present  
- `greeting`: optional generated greeting  
- `username`: chatter username  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_first_1",
  "type": "chat.first_message",
  "timestamp": "2025-02-18T21:16:00Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "memory": { "id": "mem_9", "name": "newviewer", "facts": "Loves Apex, prefers evenings", "updated": "2025-02-10T10:00:00Z" },
    "greeting": "Welcome back! How was your week?",
    "username": "newviewer",
    "sentDateTime": "2025-02-18T21:16:00Z"
  }
}
```

### ai.thoughts
- `reasoning`: human-readable reasoning  
- `shouldAnswer`: whether the AI intends to reply  
- `compiledFromMultiple`: true if aggregated from buffered thoughts  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_thoughts_1",
  "type": "ai.thoughts",
  "timestamp": "2025-02-18T21:16:05Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "reasoning": "Viewer asked for build advice; respond briefly.",
    "shouldAnswer": true,
    "compiledFromMultiple": false,
    "sentDateTime": "2025-02-18T21:16:05Z"
  }
}
```

### ai.tts.generated
- `messageId`: associated AI message id  
- `username`: AI display name  
- `textToSpeech`: text that will be rendered  
- `audioFormat`: e.g. `wav`, `mp3`  
- `generationTimeMs`: latency in ms  
- `audioSizeBytes`: size of generated audio  
- `ttsStreamUrl`: URL to the dedicated TTS SSE (requires browser key)  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_tts_1",
  "type": "ai.tts.generated",
  "timestamp": "2025-02-18T21:16:10Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "messageId": "ai_msg_1",
    "username": "ailicia",
    "textToSpeech": "I'll cover that in a moment!",
    "audioFormat": "mp3",
    "generationTimeMs": 820,
    "audioSizeBytes": 24576,
    "ttsStreamUrl": "https://api.getailicia.com/tts/{userId}/{browserKey}",
    "sentDateTime": "2025-02-18T21:16:10Z"
  }
}
```

### channel.event
- `eventType`: platform event enum (e.g. FOLLOW, SUBSCRIPTION, RAID, CHEER)  
- `username`: actor username (nullable for some events)  
- `gifter`: optional gifter username  
- `count`, `value`: numeric quantities (subs count, cheer value, etc.)  
- `raidViewers`: raid viewer count (if applicable)  
- `message`: optional message  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_channel_1",
  "type": "channel.event",
  "timestamp": "2025-02-18T21:17:00Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "eventType": "SUBSCRIPTION",
    "username": "supporter1",
    "gifter": null,
    "count": 1,
    "value": 1,
    "raidViewers": null,
    "message": "Thanks for the stream!",
    "sentDateTime": "2025-02-18T21:17:00Z"
  }
}
```

### ai.moderation
- `messageId`: moderated AI message id  
- `username`: AI display name  
- `originalMessage`: text content  
- `isAppropriate`: moderation verdict  
- `categoriesScore`: map of category => `{ flagged, score }`  
- `moderationEnabled`: moderation flag at the time  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_ai_mod_1",
  "type": "ai.moderation",
  "timestamp": "2025-02-18T21:17:10Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "messageId": "ai_msg_2",
    "username": "ailicia",
    "originalMessage": "That was rough.",
    "isAppropriate": true,
    "categoriesScore": {
      "toxicity": { "flagged": false, "score": 0.04 },
      "hate": { "flagged": false, "score": 0.01 }
    },
    "moderationEnabled": true,
    "sentDateTime": "2025-02-18T21:17:10Z"
  }
}
```

### api.event
- `eventType`: public event type  
- `content`: event payload content  
- `shouldGenerate`: whether AI should react  
- `ttl`: optional time-to-live in seconds  
- `expiresAt`: optional expiration timestamp  

```json
{
  "id": "evt_api_1",
  "type": "api.event",
  "timestamp": "2025-02-18T21:18:00Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "eventType": "CUSTOM_CALLOUT",
    "content": "Boss phase 2 is starting.",
    "shouldGenerate": true,
    "ttl": 300,
    "expiresAt": "2025-02-18T21:23:00Z"
  }
}
```

### system.join
- `channelId`: channel identifier  
- `channelName`: channel name  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_join_1",
  "type": "system.join",
  "timestamp": "2025-02-18T21:18:30Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "channelId": "ch_123",
    "channelName": "mychannel",
    "sentDateTime": "2025-02-18T21:18:30Z"
  }
}
```

### system.left
- `channelId`: channel identifier  
- `channelName`: channel name  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_left_1",
  "type": "system.left",
  "timestamp": "2025-02-18T21:19:00Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "channelId": "ch_123",
    "channelName": "mychannel",
    "sentDateTime": "2025-02-18T21:19:00Z"
  }
}
```

### character.updated
- `characterId`: active character id  
- `displayName`: character display name  
- `sentDateTime`: ISO-8601 timestamp  

```json
{
  "id": "evt_char_1",
  "type": "character.updated",
  "timestamp": "2025-02-18T21:19:30Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "characterId": "char_456",
    "displayName": "Licia Nova",
    "sentDateTime": "2025-02-18T21:19:30Z"
  }
}
```

### system.ping
- `pingId`: sequential identifier  

```json
{
  "id": "evt_ping_1",
  "type": "system.ping",
  "timestamp": "2025-02-18T21:20:00Z",
  "channel": { "id": "ch_123", "name": "mychannel", "platform": "TWITCH" },
  "payload": {
    "pingId": 123
  }
}
```
