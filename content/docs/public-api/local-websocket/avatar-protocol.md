---
title: Avatar Protocol v1
type: docs
weight: 60
prev: /docs/public-api/local-websocket/capabilities-integration-auth
next: /docs/public-api/local-websocket/avatar-events
description: Renderer-neutral protocol for speech lifecycle, mouth animation, expressions, semantic actions, audio, and recovery snapshots.
summary: Authorize, negotiate, and consume typed avatar profiles over the Desktop WebSocket.
---

`avatar.protocol.v1` carries synchronized avatar intent and optional TTS audio over the existing authenticated Desktop WebSocket. It is renderer-neutral: Unreal, browser avatars, VRM, Live2D, game engines, and physical avatars consume the same contract and keep rig-specific mappings locally.

## Runtime support

The current Desktop producer supports:

| Profile | Current output |
| --- | --- |
| `avatar.speech.lifecycle.v1` | Prepared, started, paused, resumed, and ended utterances |
| `avatar.mouth.amplitude.v1` | Normalized mouth opening synchronized to playback |
| `avatar.expression.vrm.v1` | Portable semantic expression state |
| `avatar.action.semantic.v1` | Portable and namespaced one-shot actions |
| `avatar.audio.pcm-s16le.v1` | Mono signed 16-bit little-endian PCM |
| `avatar.audio.opus.v1` | Mono raw Opus packets |

The v1 contract also reserves `avatar.mouth.viseme-15.v1` and `avatar.mouth.arkit-speech.v1`. Desktop does not currently produce those profiles, so request them as optional and inspect `rejectedProfiles`.

Expressions and speech actions are emitted only when their source can be correlated with the exact `utteranceId`, `sourceMessageId`, and `audioItemId`. The producer never guesses from text proximity or queue order.

## Connection and authorization

1. Connect to Desktop and send `hello`.
2. Read `welcome.capabilities` and require `avatar.protocol.v1`.
3. Restore or request `avatar.events.read`.
4. Send `avatar.subscribe`.
5. Read `avatar.subscribed` and use only its accepted and negotiated values.
6. Apply the immediately following `avatar.state.snapshot` before processing live state.

Use `ws://127.0.0.1:51743` on the Desktop computer. When the user enables Avatar LAN access, another device can discover `_ailicia-avatar._tcp.local` and connect to the advertised host and port. LAN connections expose only avatar authorization and avatar protocol messages.

## Subscribe and negotiate

Each profile has a `required` flag. An unsupported required profile rejects the complete subscription. Unsupported optional profiles appear in `rejectedProfiles`.

This example prefers efficient Opus over binary frames, then falls back to PCM and JSON Base64:

```json
{
  "type": "avatar.subscribe",
  "payload": {
    "requestId": "sub-72c6f988",
    "protocolVersion": 1,
    "profiles": [
      {"id": "avatar.speech.lifecycle.v1", "required": true},
      {"id": "avatar.mouth.amplitude.v1", "required": true},
      {"id": "avatar.expression.vrm.v1", "required": false},
      {"id": "avatar.action.semantic.v1", "required": false},
      {"id": "avatar.audio.opus.v1", "required": false},
      {"id": "avatar.audio.pcm-s16le.v1", "required": false}
    ],
    "speechDriverPreference": ["avatar.mouth.amplitude.v1"],
    "audioTransportPreference": ["binary-v1", "json-base64"],
    "audioFormatPreference": [
      {
        "profile": "avatar.audio.opus.v1",
        "sampleRateHz": 48000,
        "channels": 1,
        "frameDurationMs": 20,
        "bitrateBps": 24000
      },
      {
        "profile": "avatar.audio.pcm-s16le.v1",
        "sampleRateHz": 24000,
        "channels": 1
      }
    ]
  }
}
```

Preference arrays are ordered from most to least preferred. Every mouth profile requested by the client must appear exactly once in `speechDriverPreference`. Every audio format preference must refer to a requested audio profile.

The response identifies what the server selected:

```json
{
  "type": "avatar.subscribed",
  "payload": {
    "requestId": "sub-72c6f988",
    "protocolVersion": 1,
    "avatarSessionId": "avs_...",
    "subscriptionId": "asub_...",
    "acceptedProfiles": [
      "avatar.speech.lifecycle.v1",
      "avatar.mouth.amplitude.v1",
      "avatar.expression.vrm.v1",
      "avatar.action.semantic.v1",
      "avatar.audio.opus.v1",
      "avatar.audio.pcm-s16le.v1"
    ],
    "rejectedProfiles": [],
    "audioTransport": "binary-v1",
    "audioFormat": {
      "profile": "avatar.audio.opus.v1",
      "sampleRateHz": 48000,
      "channels": 1,
      "frameDurationMs": 20,
      "bitrateBps": 24000
    }
  }
}
```

If audio preferences are omitted, the server may select a compatible source PCM format. `avatar.audio.prepared` is the authoritative per-stream description, so consumers must read it even when `avatar.subscribed.audioFormat` is present.

See [Avatar Audio](/docs/public-api/local-websocket/avatar-audio) for format and binary framing details.

## Ordering and identity

Every JSON avatar event contains:

```json
{
  "schemaVersion": 1,
  "avatarSessionId": "avs_...",
  "subscriptionId": "asub_...",
  "deliverySequence": 1,
  "emittedAtMs": 1789992000123
}
```

`deliverySequence` belongs to one subscription and is assigned after authorization and profile filtering. A gap means a delivered event was missed. With `binary-v1`, binary audio chunks consume sequence numbers too. A client that observes only JSON messages will therefore see expected gaps and must process binary frames as part of the same sequence.

`utteranceId` joins speech, mouth, expression, action, and audio events for one playback. `audioItemId` joins the audio stream and exact TTS queue item. A changed `avatarSessionId` invalidates earlier transient state.

## Timing

Speech and mouth events use audio-relative integer millisecond offsets. On `avatar.speech.started` or `avatar.speech.resumed`, record a local monotonic timestamp and advance from the supplied `playbackOffsetMs`. `emittedAtMs` is diagnostic wall-clock time, not the animation clock.

The v1 playback rate is `1.0`. Evaluate late samples at the current audio-relative position or discard them. Do not accelerate animation to catch up.

## Recovery

Desktop sends `avatar.state.snapshot` immediately after a successful subscription. Request another snapshot after reconnecting or detecting a delivery gap:

```json
{
  "type": "avatar.get_state",
  "payload": {
    "requestId": "state-1",
    "subscriptionId": "asub_...",
    "lastDeliverySequence": 42
  }
}
```

The snapshot uses the next delivery sequence for that subscription. A playing snapshot re-anchors the local playback clock when received. It includes persistent expression state but never replays transient semantic actions.

## Subscription errors

Subscription failures use `avatar.subscription_error`:

```json
{
  "type": "avatar.subscription_error",
  "payload": {
    "requestId": "sub-72c6f988",
    "code": "required_profile_unsupported",
    "message": "Required profile avatar.mouth.viseme-15.v1 is not supported.",
    "profileId": "avatar.mouth.viseme-15.v1"
  }
}
```

Codes are `not_authorized`, `invalid_request`, `unsupported_protocol_version`, `required_profile_unsupported`, and `internal_error`.

## Typed contract

The [`ai_licia-client`](https://github.com/novasquare-ailicia/ai_licia-core/tree/main/packages/ai_licia-client) package is the machine-readable authority. It exports TypeScript discriminated unions, `LocalWebSocketClient`, binary audio decoding, runtime validators, and these schemas:

- [`avatar-protocol.v1.schema.json`](https://github.com/novasquare-ailicia/ai_licia-core/blob/main/packages/ai_licia-client/schema/avatar-protocol.v1.schema.json)
- [`avatar-integration-auth.v1.schema.json`](https://github.com/novasquare-ailicia/ai_licia-core/blob/main/packages/ai_licia-client/schema/avatar-integration-auth.v1.schema.json)
- [`avatar-protocol.v1.fixtures.json`](https://github.com/novasquare-ailicia/ai_licia-core/blob/main/packages/ai_licia-client/schema/avatar-protocol.v1.fixtures.json)

Run `npm run simulator:avatar` in that repository for a deterministic baseline, expression, action, and JSON PCM stream at `ws://127.0.0.1:51744`. Use Desktop for live Opus and binary delivery testing.

## Adapter boundary

Protocol identifiers describe portable intent, not renderer assets. Map expressions to morph targets, semantic actions to animations, and mouth drivers to the local rig inside the adapter. Do not interpret raw tone strings as asset names or apply amplitude, visemes, and ARKit speech coefficients simultaneously.

The [`examples/stackchan-adapter`](https://github.com/novasquare-ailicia/ai_licia-core/tree/main/examples/stackchan-adapter) example consumes this same protocol for a physical avatar. Unreal, VRM, Live2D, and browser integrations should follow the same boundary.
