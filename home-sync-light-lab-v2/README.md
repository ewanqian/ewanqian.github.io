# Open Show Sync Lab — Masked Watermark update

This replaces the audible 4-FSK transmitter in the existing browser demo with an experimental **masked spectral watermark**.

## Why it changed

The previous version proved the clock / PLL / holdover architecture, but its pure tones were audible and became unreliable when the level was reduced. This update separates the two problems:

- keep the clock discipline layer;
- replace the audible modem with a programme-dependent watermark.

## Current watermark design

The browser does **not add a beep or noise carrier**. A loaded audio file passes through 16 pairs of narrow peaking-EQ filters between about 2.6 and 9.2 kHz.

For each code pattern:

1. first 100 ms applies +pattern;
2. next 100 ms applies the complementary −pattern;
3. the receiver measures the log-energy difference of each nearby frequency pair;
4. subtracting the two halves removes much of the programme's natural spectral bias;
5. the resulting vector is correlated against known code patterns.

One frame per second:

- 0–200 ms: preamble
- 200–400 ms: minute ID 0–59
- 400–600 ms: second ID 0–59
- 600–1000 ms: untouched programme

Minute + second gives an absolute show position for programmes up to one hour.

## Receiver

The receiver runs in an AudioWorklet and uses Goertzel energy measurements instead of FFT UI polling.

Decoded anchors discipline an affine clock:

SHOW_TIME = RATE × RECEIVER_AUDIO_CLOCK + OFFSET

The local audio clock continues through short packet losses (holdover).

## Perceptual A/B test

The transmitter exposes:

- A · ORIGINAL
- B · WATERMARKED
- Δ · DIFFERENCE ×8

Start around 1.2–1.5 dB only to prove decoding, then lower the watermark toward 0.5–0.8 dB while checking whether A/B remains perceptually negligible.

## Important limitation

This is an original open experimental spectral-patchwork design. It is **not** Evixar Another Track and it does not reproduce the audiowmark algorithm. It is intentionally built as a learning prototype so the watermark transport can later be replaced without changing the clock or cue engine.

## Next steps

- improve adaptive masking based on programme energy around each frequency pair;
- add confidence-weighted PLL observations;
- add FEC / repeated IDs;
- add offline audio-file encoder/export;
- add LTC input as a reference path;
- add CSV test logging;
- port detector to ESP32-S3 + I2S microphone;
- compare acoustic sync against ESP-NOW global clock.
