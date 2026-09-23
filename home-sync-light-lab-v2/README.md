# Acoustic Sync Light Lab V2

V2 turns the first browser proof-of-concept into a learning rig for modern synchronization design.

## What changed

- Receiver DSP moved from animation-frame polling to **AudioWorklet**.
- The acoustic packet carries an **absolute show-time observation**, not LED values.
- The receiver maintains an affine clock model:

  SHOW_TIME = RATE × LOCAL_MONOTONIC_TIME + OFFSET

- Recent observations are fitted to estimate rate error / drift.
- Small clock errors are slewed by filtering the model; large errors trigger reacquisition.
- The receiver enters **HOLDOVER** if packets disappear and keeps running from its local monotonic clock.
- Diagnostics expose lock state, residual error, drift ppm, packet age, valid packets, CRC failures, sample rate, and browser output-latency estimate.
- A/B/C lighting remains local choreography so the transport can later be replaced without changing the cue engine.

## Why this is closer to a real system

The transport and timeline are separated:

1. **Transport** tells a node where the show is.
2. **Clock discipline** turns noisy observations into a smooth local show clock.
3. **Cue engine** decides what Group A/B/C should do at that time.

This is the same architecture we can keep when replacing browser 4-FSK with an audio watermark, LTC, fingerprint matching, ESP-NOW, or a wired/network time source.

## Suggested experiments

1. Baseline: two computers at 30 cm, quiet room, MID carrier.
2. Distance: 1 m, 3 m, 5 m.
3. Interference: music from the same speaker.
4. Codec path: play a screen recording / streamed copy and see whether packets survive.
5. Holdover: mute the sender for 5 s, 20 s, 60 s.
6. Profile: compare MID vs HIGH.
7. Offset: adjust acoustic offset and record residual error.

## 2026 upgrade path

### Browser layer
- AudioWorklet for real-time DSP.
- AudioContext.currentTime as the local monotonic audio clock.
- AudioContext.getOutputTimestamp() / outputLatency for output-path timing diagnostics.

### Open acoustic physical layers to study
- ggwave: compact FSK + ECC data-over-sound library.
- audiowmark: blind audio watermarking using a spectral patchwork approach.
- browser 4-FSK projects with AudioWorklet, framing, CRC/ACK and modem tests.
- libltc / LTC.wasm: standard SMPTE LTC interop.

### Timeline recovery
Use two complementary modes:
- **Watermark/timecode** for precise absolute show position.
- **Fingerprint** for recovery / relocation when the watermark is unavailable.

### Hardware layer
Recommended next node:
- ESP32-S3
- I2S microphone such as INMP441
- local cue table
- addressable LEDs
- ESP-NOW as a parallel RF clock / provisioning path

The current ESP-NOW SDK includes internal time-synchronization APIs. This is useful for a hybrid design where acoustic sync keeps a node aligned with the sound it hears, while RF provides a global simultaneous clock for large fields.

### Musical-network baseline
Ableton Link is worth keeping as a separate comparison path. It synchronizes tempo, beat and phase across peers, but intentionally does not impose one identical absolute media timeline. This makes it useful for VJ / music interaction, but different from absolute show-time recovery.

## V3 target

- packet confidence and preamble correlation
- Reed-Solomon or convolutional FEC + interleaving
- automatic constant-delay calibration
- LTC receive mode
- WebSocket/UDP bridge for Resolume / TouchDesigner / Processing
- ESP32-S3 receiver firmware
- experiment logger / CSV export
- network-vs-acoustic A/B comparison
