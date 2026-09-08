# Mirroring over a USB cable

USB mirroring uses Apple's screen-capture interface — the same one
QuickTime uses on a Mac — rather than AirPlay. It needs no network.

**On a Raspberry Pi 4 this is not yet usable.** See "Performance" below.
On an x86 PC it runs at about 50 ms.

## Steps

1. Connect the iPhone with a USB **data** cable (charge-only cables do
   not enumerate).
2. Unlock the phone and tap **Trust This Computer**; you will need the
   passcode. Each computer is trusted separately.
3. On the **Home** page, turn **USB Cable** on under Mirror Source.
4. Click **Start**.

A second window, `rPlay USB`, opens with the phone's screen.

## First-time pairing needs a replug

If this is the first time the phone has trusted this machine, mirroring
does not begin the moment you tap Trust — **unplug and plug the cable
back in**. The pairing completes on the first connection and the mirror
starts on the next one. Once paired, it comes up on plug-in.

## Performance

Measured on a Pi 4 with an iPhone 12:

| | |
|---|---|
| Latency | ~480 ms |
| Frame rate | 19 fps |
| CPU | ~140 % of one core |

The cable is not the problem — the same code and the same phone on an
x86 laptop give 50 ms. USB delivers a near-native 888×1920 stream, 2.9×
the pixels of AirPlay's 534×1080, and a Cortex-A72 cannot decode that in
software at 60 fps. The shortfall accumulates and shows up as latency.

Hardware H.264 decoding on the Pi would fix this and does not work yet.

## Audio

USB-mirror audio needs the daemon to reach your sound server. Running it
as root with a desktop session's `XDG_RUNTIME_DIR` does not work — the
log fills with `XDG_RUNTIME_DIR ... is not owned by us`. Video is
unaffected.
