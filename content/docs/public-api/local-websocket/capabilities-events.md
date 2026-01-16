---
title: "Capability: Events"
type: docs
weight: 30
prev: /docs/public-api/local-websocket/messages
next: /docs/public-api/local-websocket/capabilities-state
description: Event messages emitted by the local WebSocket.
summary: Details for the events capability and the ttsAmplitude event payload.
---

The `events` capability means the local WebSocket can push event notifications to integration clients.

## Event envelope
Events are sent with a generic envelope:

```json
{
  "type": "event",
  "payload": {
    "name": "ttsAmplitude",
    "data": {}
  }
}
```

Fields:
- `payload.name`: event name string
- `payload.data`: event-specific payload

## Event: ttsAmplitude
Emitted while TTS is active to help drive lipsync or visualizers.

```json
{
  "type": "event",
  "payload": {
    "name": "ttsAmplitude",
    "data": {
      "soundAmplitudeValue": 0.3,
      "animationId": "abc123"
    }
  }
}
```

Payload fields:
- `soundAmplitudeValue`: number in the range `0..1`
- `animationId`: optional identifier for the current animation (string or number)

Notes:
- The event is only delivered to clients that completed the `hello` handshake with `clientType: "integration"`.
- Expect frequent updates while speech is active; debounce or smooth on your side if needed.
