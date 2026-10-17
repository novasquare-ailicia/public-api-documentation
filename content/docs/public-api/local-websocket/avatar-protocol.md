---
title: Avatar Protocol v1
type: docs
weight: 60
prev: /docs/public-api/local-websocket/capabilities-integration-auth
next: /docs/public-api/rest
description: Renderer-neutral local protocol for speech lifecycle, mouth animation, expressions, semantic actions, and recovery snapshots.
summary: Subscribe to typed avatar animation profiles over the authenticated Desktop WebSocket.
---

`avatar.protocol.v1` publishes renderer-neutral animation intent over the existing authenticated local WebSocket. Unreal Engine is the first external validation target, but the protocol also supports browser, VRM, Live2D, Unity, Godot, and other adapters.

{{< callout context="info" title="Initial Desktop producer" >}}
The Desktop producer supports `avatar.speech.lifecycle.v1`, `avatar.mouth.amplitude.v1`, `avatar.expression.vrm.v1`, and `avatar.action.semantic.v1`. Expressions, reaction actions, and physical gestures are emitted only when a structured speech plan matches the exact message and audio item. A known TTS tone supplies an expression fallback when no matching plan is available. Visemes and ARKit speech coefficients remain optional future v1 profiles.
{{< /callout >}}

## Baseline flow

1. Connect to `ws://127.0.0.1:51743` and send `hello`.
2. Obtain or restore the `avatar.events.read` scope.
3. Send `avatar.subscribe` with lifecycle and amplitude profiles.
4. Receive `avatar.subscribed` followed immediately by `avatar.state.snapshot` with `deliverySequence: 1`.
5. Drive the mouth only from the utterance's declared `activeSpeechDriver`.
6. Request a new snapshot after a delivery gap or reconnect.

## Subscribe

```json
{
  "type": "avatar.subscribe",
  "payload": {
    "requestId": "sub-72c6f988",
    "protocolVersion": 1,
    "profiles": [
      {"id": "avatar.speech.lifecycle.v1", "required": true},
      {"id": "avatar.mouth.amplitude.v1", "required": true},
      {"id": "avatar.expression.vrm.v1", "required": false}
    ],
    "speechDriverPreference": ["avatar.mouth.amplitude.v1"]
  }
}
```

The server rejects the complete subscription when a required profile is unsupported. Unsupported optional profiles are reported in `rejectedProfiles`.

## Ordering

Every avatar event contains:

```json
{
  "schemaVersion": 1,
  "avatarSessionId": "avs_...",
  "subscriptionId": "asub_...",
  "deliverySequence": 1,
  "emittedAtMs": 1789992000123
}
```

`deliverySequence` is scoped to one subscription and increments only for events delivered to that subscription. Events filtered out by profile selection do not consume a number. A changed `avatarSessionId` invalidates earlier transient state.

## Profiles

| Profile | Purpose |
| --- | --- |
| `avatar.speech.lifecycle.v1` | Preparation, playback, pause, resume, and end state |
| `avatar.mouth.amplitude.v1` | Normalized real-time mouth opening |
| `avatar.mouth.viseme-15.v1` | Optional portable timed viseme timeline |
| `avatar.mouth.arkit-speech.v1` | Optional ARKit-named mouth and jaw coefficients |
| `avatar.expression.vrm.v1` | Optional renderer-neutral semantic expression state |
| `avatar.action.semantic.v1` | Optional one-shot semantic animation triggers |

Exactly one mouth profile is the active speech driver for an utterance. Renderers must not apply amplitude, visemes, and ARKit speech frames simultaneously.

## Speech performance

When a structured speech plan matches the exact message, Desktop maps its dominant delivery to a portable semantic expression. The expression source is `speech_delivery` and carries the exact `utteranceId`, `sourceMessageId`, `audioItemId`, source segment index, and mapping version.

A reaction or physical gesture at the beginning of the utterance may also emit one one-shot semantic action. Audible reactions use a `speech_reaction` source. Silent gestures use a `speech_gesture` source with one of `wink`, `nod`, `shake_head`, or `rage`. The source carries the same exact utterance, message, audio item, segment, and mapping identities.

One opening segment cannot request both a reaction and a gesture. Cues in later segments are not approximated because they require aligned TTS cue timing before they can be emitted reliably. Standard action identifiers such as `nod` and `shake_head` are portable. Renderer-specific additions such as `ai.licia/wink` and `ai.licia/rage` are namespaced and may be ignored by renderers that do not implement them.

Raw provider tags, model tone strings, Unreal asset names, and robot commands are never exposed as protocol identifiers. Every renderer keeps those mappings locally.

## Timing

Speech events use audio-relative integer millisecond offsets. On `started` or `resumed`, record a local monotonic timestamp and advance from the supplied `playbackOffsetMs`. `emittedAtMs` is diagnostic wall-clock time and is not the animation clock.

The v1 playback rate is fixed at `1.0`. Late samples are discarded or evaluated at their current timeline position; renderers must not speed up animation to catch up.

## Recovery

Send `avatar.get_state` with the active `subscriptionId` after a sequence gap:

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

The returned snapshot uses the next contiguous delivery sequence. A playing snapshot re-anchors the receiver clock when it is received.

## Typed contract and simulator

The `ai_licia-client` package exports the complete TypeScript contract, runtime validators, and generated JSON Schemas:

- `schema/avatar-protocol.v1.schema.json`
- `schema/avatar-integration-auth.v1.schema.json`
- `validateAvatarProtocolMessage`
- `validateAvatarProtocolSequence`
- `LocalWebSocketClient`

The source repository also includes a local WebSocket simulator on `ws://127.0.0.1:51744` for adapter development before connecting to Desktop.

## StackChan factory-firmware example

The [`examples/stackchan-adapter`](https://github.com/novasquare-ailicia/ai_licia-core/tree/main/examples/stackchan-adapter) example consumes the same protocol and controls a stock M5Stack StackChan over BLE. It supports simulated output, mouth amplitude, expression geometry, side LEDs, conservative semantic head movements, Wink, Nod, Shake Head, and Rage without flashing custom firmware. The factory mouth can open, move, and rotate but cannot curve into a smile, so Happy is expressed through its eyes, lighting, and posture.
