[← Back to rPlay for Linux and Raspberry Pi](../README.md)

# rPlay — help

An AirPlay receiver for Linux and the Raspberry Pi: mirror an iPhone or
iPad to your screen over Wi-Fi, or over a USB cable, and drive the phone
back from your mouse and keyboard.

| | |
|---|---|
| **[raspberry-pi.md](raspberry-pi.md)** | **Complete Raspberry Pi guide — start here** |
| [install.md](install.md) | Installing, and what the first run asks for |
| [airplay.md](airplay.md) | Mirroring over Wi-Fi |
| [usb-mirroring.md](usb-mirroring.md) | Mirroring over a USB cable |
| [headless.md](headless.md) | Running on boot, without a desktop |
| [troubleshooting.md](troubleshooting.md) | When something does not work |

## What to expect on a Raspberry Pi 4

Measured on a Pi 4 (4 × Cortex-A72 @ 1.5 GHz), iPhone 12, software
decoding:

| | AirPlay (Wi-Fi) | USB cable |
|---|---|---|
| Latency | **40–60 ms** | ~480 ms |
| Frame rate | 29–41 fps | 19 fps |
| CPU | ~100 % of one core (of four) | ~140 % |

**AirPlay on a Pi 4 is good. USB mirroring on a Pi 4 is not yet.** The
reason is resolution: AirPlay negotiates a downscaled 534×1080 stream,
while USB delivers a near-native 888×1920 — 2.9× the pixels. A Pi decodes
the first comfortably and cannot keep up with the second, and the backlog
is the latency. On an x86 desktop, USB mirroring runs at 50 ms.

If you want USB mirroring today, use it on a PC. On a Pi, use AirPlay.
