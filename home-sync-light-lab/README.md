# Acoustic Sync Light Lab

A browser-based acoustic timecode experiment inspired by the **system concept** of Rhizomatiks + Evixar Home Sync Light. It does **not** implement or reverse engineer Evixar Another Track®. The purpose is to test an open, low-cost architecture: a show clock is encoded into sound, recovered by a microphone, and used to run a local pre-programmed light cue engine.

## Browser test

Open the page on two computers:

1. Computer A → **TRANSMITTER** → Start + Audio.
2. Computer B → **RECEIVER** → Enable Microphone.
3. Place B near A's speaker. Do not use headphones.
4. Use the same carrier profile on both machines.
5. When the receiver reports **LOCKED**, compare the large timecode and the virtual A/B/C light groups.

The page uses alternating-bank 4-FSK. Each frame contains:
- 6-symbol preamble
- 15-bit absolute timeline tick at 0.25 s resolution (supports ~68 min)
- 1 RUN/PAUSE bit
- CRC-8

A receiver extrapolates the show clock locally between absolute sync packets.

## Why this architecture scales

Do not try to stream every LED value through sound. Broadcast only **global show time / cue state**. Each receiver stores the same light program and has a local identity such as Group A, B, or C.

```
show master
  └─ acoustic timecode embedded in playback
       ├─ node A01 → group=A → local cue table → LEDs
       ├─ node B07 → group=B → local cue table → LEDs
       └─ node C12 → group=C → local cue table → LEDs
```

This lets hundreds of nodes react differently while receiving the same tiny signal.

## Existing hardware

### Practice Board + Pro Micro / ATmega32U4
Useful now as:
- six show buttons
- rotary master control
- four onboard SK6812MINI-E LEDs as miniature fixtures
- USB MIDI controller
- future acoustic receiver after adding an amplified microphone / envelope or conditioned analog input

The board's stock QMK definition places its four addressable LEDs on ATmega32U4 PD3 (Arduino Pro Micro TXO/D1). Its TRRS connector is **not a ready-made microphone input**.

### D1 Mini Pro / ESP8266
Useful now as:
- Wi-Fi cue bridge
- UDP / OSC / Art-Net experiment node
- external addressable LED driver

It can decode simple audio with an analog front-end, but its single ADC makes it a less comfortable acoustic receiver platform than ESP32-S3 + I2S microphone.

## Recommended next receiver node

For a reproducible receiver:
- ESP32-S3 development board
- INMP441 I2S MEMS microphone
- WS2812B/SK6812 for short 5 V prototypes, or WS2815 12 V for longer field runs
- 74AHCT125/74HCT level shifter for robust LED data
- separate LED power rail, common ground, fuse, bulk capacitor
- receiver group ID stored in flash / DIP switch / provisioning page

## Direction and zones

A single microphone listening to one broadcast acoustic code gives **time**, not direction or physical position. A/B/C grouping should therefore normally be configured in each node.

If separate acoustic transmitters are used per zone, spill between zones becomes a design problem. For a large field, a hybrid system is better:
- acoustic code = compatibility / local sound sync / fallback
- ESP-NOW, Wi-Fi multicast or another RF clock = global simultaneous sync
- local firmware = actual choreography

Sound propagation is about 343 m/s at room temperature, so acoustic-only global synchronization accumulates about 2.9 ms of delay per metre. At home this is usually small; across 100 m it becomes roughly 0.29 s. Acoustic sync can still keep a lamp aligned to the **sound heard at its own position**, but it cannot make a very large field globally simultaneous without another clock path.

## Development path

V0: two-browser acoustic sync (this page)  
V1: Practice Board 4-LED receiver  
V2: ESP32-S3 + INMP441 + 1 strip  
V3: A/B/C field nodes with stored cue program  
V4: hybrid acoustic + ESP-NOW clock  
V5: Resolume / Ableton / OBS transmitter bridge and hardware show recorder
