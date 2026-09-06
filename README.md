# rPlay for Linux and Raspberry Pi

rPlay turns a Linux machine into an **AirPlay 2 receiver**: an iPhone or iPad
mirrors its screen to it over Wi-Fi, and video flung from apps such as YouTube
plays back natively.

This repository is for **issues, screenshots and release notes only**.
The source code is not public.

## Raspberry Pi

Built and running natively on **Raspberry Pi OS Bookworm (64-bit)**, verified
on a Raspberry Pi 4 Model B. The same 64-bit image and build also target the
Pi 3 and Pi 5.

![rPlay mirroring an iPhone on a Raspberry Pi 4](images/rplay-rpi4-fullscreen.png)

*The whole Pi desktop: the rPlay window, the live iPhone mirror at 534×1080,
and the tool rail for pin, home, screenshot, record and input.*

![The rPlay window while mirroring](images/rplay-rpi4-mirroring.png)

*The receiver window on its own.*

![rPlay idle on Raspberry Pi OS](images/rplay-rpi4-idle.png)

*Idle, advertising itself over Bonjour as an AirPlay receiver.*

### What it does today

- AirPlay 2 screen mirroring from iPhone and iPad
- Video streaming from apps (YouTube and similar), decoded on the Pi
- AirPlay audio
- Bonjour advertising through Apple's mDNSResponder
- A small control sidebar, with optional Bluetooth HID input to the phone

### Built on the Pi from

| Component | Version |
|---|---|
| Raspberry Pi OS | Bookworm, 64-bit |
| FFmpeg | 3.4, aarch64 with NEON |
| SDL2 | 2.0.22, X11 and KMSDRM |
| OpenSSL | 1.1.1w |
| fdk-aac | 2.0.2 |
| Bonjour | Apple mDNSResponder |

A full native build takes roughly 100 minutes on a Pi 4, most of it FFmpeg.
The result is a single ~20 MB `aarch64` binary, packaged as an `arm64` `.deb`.

## Reporting a problem

Open an issue and include:

1. Pi model and RAM, and the output of `cat /etc/os-release`
2. iPhone or iPad model and iOS version
3. What you did and what happened
4. The receiver log, with any addresses removed

## Status

Early. Mirroring and video playback work on the Pi; frame rate on a Pi 4 with
software decode is the main thing being improved.
