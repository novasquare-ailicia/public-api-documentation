---
title: Avatar Event Reference
type: docs
weight: 70
prev: /docs/public-api/local-websocket/avatar-protocol
next: /docs/public-api/local-websocket/avatar-audio
description: Event reference for avatar speech lifecycle, mouth animation, expressions, semantic actions, and recovery state.
summary: Apply each typed event to the correct animation layer and clock.
---

Avatar events use the normal WebSocket event envelope:

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.mouth.amplitude",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 8,
      "emittedAtMs": 1789992000123
    }
  }
}
```

All identifiers are opaque. Do not infer relationships from their textual form.

## Speech lifecycle

| Event | Additional fields | Meaning |
| --- | --- | --- |
| `avatar.speech.prepared` | `utteranceId`, optional `sourceMessageId`, optional `audioItemId`, optional `durationMs`, optional `toneId`, optional `characterId`, `activeSpeechDriver`, `availableProfiles` | Creates the utterance and declares its selected mouth driver |
| `avatar.speech.started` | `utteranceId`, `playbackOffsetMs` | Audio playback became audible |
| `avatar.speech.paused` | `utteranceId`, `playbackOffsetMs`, `reason` | Playback paused |
| `avatar.speech.resumed` | `utteranceId`, `playbackOffsetMs` | Playback resumed |
| `avatar.speech.ended` | `utteranceId`, `playbackOffsetMs`, `reason` | Playback is no longer active |

Pause reasons are `speaker_interruption`, `user_pause`, and `system_pause`. End reasons are `completed`, `discarded`, `interrupted`, `playback_error`, `account_changed`, and `shutdown`.

Use `started`, not `prepared`, as the audible playback anchor. Re-anchor on `resumed` and on an active snapshot.

## Mouth layer

### Amplitude

`avatar.mouth.amplitude.v1` emits:

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.mouth.amplitude",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 8,
      "emittedAtMs": 1789992000123,
      "utteranceId": "utt_...",
      "offsetMs": 160,
      "value": 0.64
    }
  }
}
```

`value` is normalized to `0..1`. Smooth it locally if the rig needs damping, but retain `offsetMs` as the alignment source.

### Visemes and ARKit speech

The v1 contract defines `avatar.mouth.visemes` and `avatar.mouth.arkit_frames` as revisioned, chunked timelines. They are optional profiles and are not currently produced by Desktop.

A renderer may request several mouth profiles for negotiation, but applies only the utterance's `activeSpeechDriver`. Mouth profiles never control blinking, gaze, eyebrows, semantic expression, or body pose.

## Semantic expression layer

`avatar.expression.state` replaces the complete semantic expression layer. It is state, not a transient cue:

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.expression.state",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 9,
      "emittedAtMs": 1789992000130,
      "revision": 12,
      "source": {
        "kind": "speech_delivery",
        "deliveryId": "angry",
        "utteranceId": "utt_...",
        "sourceMessageId": "msg_...",
        "audioItemId": "audio_...",
        "segmentIndex": 0,
        "mappingVersion": "speech-plan.v1"
      },
      "transitionMs": 120,
      "values": [
        {"expression": "angry", "weight": 0.9}
      ]
    }
  }
}
```

Standard expressions are `neutral`, `happy`, `angry`, `sad`, `relaxed`, and `surprised`. Weights are normalized to `0..1`, expression identifiers cannot repeat, and a newer `revision` replaces the previous state.

The expression remains active until a newer expression event replaces it. Current Desktop behavior may keep the latest emotion after speech and later transition toward neutral, but hold duration is producer policy, not a protocol guarantee.

Blend the semantic expression beside the active mouth layer. Renderer-local blink and gaze remain separate.

## Semantic action layer

`avatar.action.triggered` is a one-shot intent:

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.action.triggered",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 10,
      "emittedAtMs": 1789992000140,
      "actionInstanceId": "action_...",
      "actionId": "nod",
      "intensity": 0.8,
      "durationMs": 500,
      "source": {
        "kind": "speech_gesture",
        "gestureId": "nod",
        "utteranceId": "utt_...",
        "sourceMessageId": "msg_...",
        "audioItemId": "audio_...",
        "segmentIndex": 0,
        "mappingVersion": "speech-plan.v1"
      }
    }
  }
}
```

Standard action identifiers are `wave`, `nod`, `shake_head`, `celebrate`, `laugh`, and `surprised_recoil`. Namespaced actions such as `ai.licia/wink` may be ignored by adapters that do not implement them.

`actionInstanceId` deduplicates one trigger. Actions may temporarily override local pose or expression, but the adapter must restore current state afterward. Snapshots do not replay actions.

## State snapshots

`avatar.state.snapshot` contains:

- The profiles selected for this subscription.
- The currently playing or paused utterance, or `null`.
- The current semantic expression when that profile was selected.
- An optional `requestId` when answering `avatar.get_state`.

Snapshots are client-specific. Their `deliverySequence` continues the subscription sequence, including any binary audio chunks already delivered.

## Layer precedence

1. Apply exactly one negotiated speech driver to the mouth.
2. Blend semantic expression on the expression layer.
3. Keep local blink and gaze independent.
4. Let a semantic action temporarily override only the local layers required by that action.
5. Restore the current protocol state after the action completes.

Renderer asset names, bone names, morph target names, robot commands, and animation montage identifiers stay inside the adapter.
