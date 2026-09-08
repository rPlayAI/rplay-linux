# Mirroring over Wi-Fi (AirPlay)

1. Put the Pi and the iPhone on the same network.
2. Start rPlay and click **Start**.
3. On the iPhone, open Control Centre → **Screen Mirroring**, and pick the
   name shown in rPlay (default `rPlay`).

A window opens with the phone's screen. Mouse, keyboard and touch on that
window are forwarded back to the phone.

## Performance

On a Raspberry Pi 4, measured: **40–60 ms** of latency at 29–41 fps, for
about one core of the four. That is roughly four times quicker than
UxPlay on the same board, because rPlay hands each decoded frame straight
to the renderer rather than through a buffering pipeline.

AirPlay negotiates a downscaled stream — 534×1080 from an iPhone 12 in
portrait — which is why a Pi keeps up with it comfortably in software.

## Audio

Audio follows the mirror. Pick the output in `/etc/rplay/conf`:

```sh
RPLAY_AUDIO_DEVICE=hw:0      # HDMI (typical)
RPLAY_AUDIO_DEVICE=plughw:2  # the 3.5 mm jack
```

Without it, ALSA's `default` device is used, which hands the choice to
PipeWire — often the headphone jack even when your screen is on HDMI.

## Video and YouTube

Sending a video from the YouTube app (the AirPlay icon inside the app,
rather than screen mirroring) plays it directly on the Pi rather than
mirroring it, so it runs at full quality. iQiyi works the same way; press
the app's own Play button once the window appears.
