---
title: Avatar Audio
type: docs
weight: 80
prev: /docs/public-api/local-websocket/avatar-events
next: /docs/public-api/rest
description: Negotiate PCM or Opus TTS audio using JSON Base64 or compact binary WebSocket frames.
summary: Select an audio format independently from its WebSocket transport and keep it aligned with avatar events.
---

Audio is an optional Avatar Protocol profile. It uses the same authorization, subscription, `utteranceId`, `audioItemId`, and `deliverySequence` as speech and animation events. Do not open a second socket.

## Formats

| Profile | Encoding | Sample rates | Channels | Additional settings |
| --- | --- | --- | --- | --- |
| `avatar.audio.pcm-s16le.v1` | Signed 16-bit little-endian PCM | 24000, 44100, 48000 Hz | 1 | None |
| `avatar.audio.opus.v1` | Raw Opus packets | 24000, 48000 Hz | 1 | 20, 40, or 60 ms frames; 6000 to 128000 bit/s |

Opus payloads are individual raw Opus packets. They are not Ogg or WebM files and contain no container headers.

## Transports

| Transport | Chunk representation | Use when |
| --- | --- | --- |
| `binary-v1` | Binary WebSocket message | Efficiency and low decoding overhead matter |
| `json-base64` | `avatar.audio.chunk` JSON event with `dataBase64` | Simplicity or text-only WebSocket tooling matters |

Format and transport are independent. For example, Opus may be carried in Base64 JSON and PCM may be carried in a binary frame.

If `audioTransportPreference` is omitted, Desktop defaults to `json-base64`. For predictable output, send both ordered preference arrays and read `avatar.subscribed.audioTransport` and `avatar.subscribed.audioFormat`.

## Stream lifecycle

Both audio profiles use the same three event names:

1. `avatar.audio.prepared` declares the final stream format.
2. Zero or more chunks carry ordered encoded audio.
3. `avatar.audio.completed` declares the final counts.

### Prepared

PCM example:

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.audio.prepared",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 3,
      "emittedAtMs": 1789992000100,
      "utteranceId": "utt_...",
      "audioItemId": "audio_...",
      "encoding": "PCM_S16_LE",
      "sampleRateHz": 24000,
      "channels": 1,
      "durationMs": 2500
    }
  }
}
```

Opus additionally carries `encoding: "OPUS"`, `frameDurationMs`, and `bitrateBps`.

### JSON Base64 chunks

With `json-base64`, each chunk is a normal avatar event:

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.audio.chunk",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 4,
      "emittedAtMs": 1789992000110,
      "utteranceId": "utt_...",
      "audioItemId": "audio_...",
      "chunkIndex": 0,
      "sampleOffset": 0,
      "dataBase64": "..."
    }
  }
}
```

For PCM, Base64 decodes directly to little-endian signed 16-bit samples. For Opus, it decodes to one raw Opus packet.

### Completed

```json
{
  "type": "event",
  "payload": {
    "name": "avatar.audio.completed",
    "data": {
      "schemaVersion": 1,
      "avatarSessionId": "avs_...",
      "subscriptionId": "asub_...",
      "deliverySequence": 42,
      "emittedAtMs": 1789992002600,
      "utteranceId": "utt_...",
      "audioItemId": "audio_...",
      "chunkCount": 38,
      "totalSamples": 120000
    }
  }
}
```

`chunkIndex` starts at zero. `sampleOffset` and `totalSamples` count decoded mono samples at the negotiated sample rate, including Opus frame padding when present.

## binary-v1 frame

With `binary-v1`, prepared and completed remain JSON events. Each audio chunk becomes one binary WebSocket message with this big-endian header:

| Offset | Size | Field | Value |
| ---: | ---: | --- | --- |
| 0 | 4 | Magic | ASCII `AVP1` |
| 4 | 1 | Frame type | `1` for audio chunk |
| 5 | 1 | Frame version | `1` |
| 6 | 2 | Flags | `0` |
| 8 | 8 | Delivery sequence | Unsigned 64-bit integer |
| 16 | 4 | Chunk index | Unsigned 32-bit integer |
| 20 | 8 | Sample offset | Unsigned 64-bit integer |
| 28 | 2 | Audio item ID length | Unsigned 16-bit byte count |
| 30 | variable | Audio item ID | UTF-8 bytes |
| following | remaining | Audio payload | PCM bytes or one Opus packet |

The frame does not repeat `utteranceId` or the encoding. Resolve its `audioItemId` against the preceding `avatar.audio.prepared` event and the subscription's negotiated format.

Binary chunks consume the same contiguous `deliverySequence` as JSON avatar events. JavaScript numbers can represent protocol sequence and offset values only while they remain safe integers; the official decoder rejects larger values.

The `ai_licia-client` SDK exposes `decodeAvatarBinaryAudioFrame` and dispatches valid frames through `LocalWebSocketClient.onBinaryAudio`:

```typescript
client.onBinaryAudio((frame) => {
  audioQueue.push({
    audioItemId: frame.audioItemId,
    chunkIndex: frame.chunkIndex,
    sampleOffset: frame.sampleOffset,
    encodedBytes: frame.data
  });
});
```

## Playback guidance

- Buffer enough audio for the selected device before starting playback.
- Treat `avatar.speech.started` as the audible playback anchor, not the arrival time of the first chunk.
- Keep audio and mouth events keyed by the same `utteranceId`.
- Preserve chunk order and detect delivery gaps before decoding.
- On reconnect, start a new subscription. Snapshots recover avatar state, not earlier audio chunks.
- Never play both Desktop audio and renderer audio unless the user explicitly wants duplicated output.
