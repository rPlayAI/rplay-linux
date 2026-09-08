# rPlay on a Raspberry Pi — complete guide

Mirror an iPhone or iPad to a Raspberry Pi over Wi-Fi or a USB cable, and
drive the phone from the Pi's mouse and keyboard.

This page is the long version. For a quick start, read
[install.md](install.md).

---

## 1. What to expect

Measured on a Raspberry Pi 4 (4 × Cortex-A72 @ 1.5 GHz) with an iPhone 12,
software decoding:

| | AirPlay (Wi-Fi) | USB cable |
|---|---|---|
| Latency | **40–60 ms** | ~480 ms |
| Frame rate | 29–49 fps | 19 fps |
| CPU | ~1 core of 4 | ~1.4 cores of 4 |
| Memory | ~250 MB | ~165 MB |

**Use AirPlay on a Pi.** USB mirroring works but is not pleasant there: USB
delivers a near-native 888×1920 stream, 2.9× the pixels of AirPlay's
534×1080, and a Cortex-A72 cannot software-decode that at 60 fps. The
shortfall accumulates and shows up as latency. The same code on an x86 PC
does USB mirroring at 50 ms.

## 2. Requirements

- Raspberry Pi 4 (2 GB or more) running Raspberry Pi OS Bookworm, 64-bit.
- A display on HDMI, and a desktop session — rPlay opens windows.
- The Pi and the iPhone on the same network. 5 GHz is worth preferring.
- For **control features**: see §5.

A Pi 5 has no hardware H.264 decoder and has not been tested. A Pi 3 has
not been tested and will likely be short of both CPU and memory.

## 3. Installing

```sh
sudo apt-get install -y ./rplay_0.4.1_arm64.deb
```

| Installed | |
|---|---|
| `/opt/rplay/bin/rplay` | the receiver |
| `/opt/rplay/bin/rplay_wd` | USB-mirroring daemon |
| `/opt/rplay/bin/mdnsd` | Bonjour (Apple's own) |
| `/opt/rplay/bin/usb-host-services.sh` | frees the phone from usbmuxd/GVFS |
| `/etc/rplay/conf` | settings, all optional |

Launch from the applications menu, or run `/opt/rplay/bin/rplay`.

### Bonjour and avahi

rPlay ships Apple's `mdnsd` as `rplay-mdnsd`. It and `avahi-daemon` both
bind UDP 5353, so the unit declares `Conflicts=avahi-daemon` — systemd
stops avahi while rPlay's is running and lets it back afterwards. You do
not need to disable avahi yourself.

## 4. Mirroring over Wi-Fi

1. Start rPlay and click **Start**. rPlay does not advertise on the network
   until you do — this is deliberate.
2. On the iPhone: Control Centre → **Screen Mirroring** → pick the name
   rPlay shows — `rPlay` unless you renamed it.

A window opens with the phone's screen. Closing it ends the session.

### The password prompt on first Start

The first time you click Start, rPlay asks for your password once.

This is not rPlay wanting root for itself. A stock Linux desktop grabs an
iPhone the moment it is plugged in: `usbmuxd` claims the lockdown
interface, and the GVFS volume monitors open the device to enumerate it —
which is why the phone often appears as a folder. While they hold it,
rPlay cannot open it at all.

rPlay clears them once and remembers that it asked. To undo it, the
**AirPlay** and **USB** tabs both have a **Restore desktop iPhone
handling** button. Or by hand:

```sh
/opt/rplay/bin/usb-host-services.sh status     # what is in the way
/opt/rplay/bin/usb-host-services.sh disable    # clear it
/opt/rplay/bin/usb-host-services.sh enable     # put it back
```

## 5. Control features

Alongside mirroring, you can drive the phone from the machine's mouse and
keyboard — click, scroll and type on the mirror window and the phone responds.

Two things to set up first:

1. **Pair the iPhone with the Pi** from the Pi's Bluetooth settings.
2. **On the phone, turn on Settings → Accessibility → Zoom.**
   (AssistiveTouch also works.) Without one of these, iOS ignores the
   forwarded touches.

rPlay reminds you about step 2 the first time control becomes active. Once you
know, set `RPLAY_IAP_TOUCH_NO_REMINDER=1` in `/etc/rplay/conf` to stop the
reminder.

If control is not available, mirroring, audio and video fling are unaffected —
only mouse, keyboard and touch input are.

## 6. Mirroring over a USB cable

See [usb-mirroring.md](usb-mirroring.md) for detail. In short: use a data
cable, tap **Trust This Computer**, turn **USB Cable** on under Mirror
Source on the Home page, then Start.

**First-time pairing needs a replug.** Pairing completes on one connection
and mirroring starts on the next, so unplug and reconnect after tapping
Trust.

## 7. Running on boot

```
--autostart, -a      start the receiver at launch, no click needed
--usb-mirror         also bring up the USB mirror (implies --autostart)
--no-host-prepare    skip clearing usbmuxd / the GVFS monitors
```

`--no-host-prepare` matters unattended — clearing those needs root and
nothing can answer a password dialog. Prepare the host once by hand:

```sh
sudo /opt/rplay/bin/usb-host-services.sh disable
```

A systemd unit is in [headless.md](headless.md). rPlay needs a display, so
it belongs to `graphical.target`, not `multi-user.target`.

## 8. Settings — `/etc/rplay/conf`

| Variable | Purpose |
|---|---|
| `RPLAY_AUDIO_DEVICE` | ALSA output. `hw:0` = HDMI, `plughw:2` = 3.5 mm jack |
| `RPLAY_NAME` | Name in the iPhone's AirPlay list. A name set in the GUI wins; with neither it is `rPlay` |
| `RPLAY_PROXY_IDLE_MS` | Segment-fetch timeout. Raise to 2500 on slow Wi-Fi |
| `RPLAY_USB_MIRROR` | Start with the USB mirror enabled |
| `RPLAY_IAP_TOUCH_NO_REMINDER` | Stop the control reminder dialog |
| `RPLAY_SW_RENDER` | `0` forces the accelerated renderer |
| `RPLAY_H264_DECODER` | Hardware decode opt-in — **not usable yet**, see §10 |

Audio note: without `RPLAY_AUDIO_DEVICE`, ALSA's `default` defers to
PipeWire, which often picks the headphone jack even with the screen on
HDMI.

## 9. Troubleshooting

See [troubleshooting.md](troubleshooting.md) for the full list. The three
most common:

- **Phone appears as a folder and rPlay sees nothing** — the desktop
  grabbed it. Run `usb-host-services.sh disable`, then replug.
- **Phone does not see rPlay** — click Start; rPlay is silent until you do.
  Check `rplay-mdnsd` owns 5353: `sudo ss -lunp | grep 5353`.
- **Control does nothing** — see §5.

## 10. Known issues

- **Hardware H.264 decode does not work.** `h264_v4l2m2m` runs but at
  ~2.6 fps against 19 fps for software decode, and saves no CPU. Do not
  set `RPLAY_H264_DECODER`. Upstream ffmpeg cannot drive this decoder
  usefully; Raspberry Pi's patched build is required and still not enough.
- **USB mirroring is slow on a Pi** (~480 ms). Use AirPlay.
- **Occasional brief hiccup.** A ~400 ms stall appears roughly once every
  few minutes in some sessions. Cause not yet identified; it is not
  reproducible on demand and most sessions are clean.
- **Reconnection after unplugging USB can leave the mirror dead.** Replug
  again, or restart rPlay.
- **USB-mirror audio does not route** when the daemon runs as root with a
  desktop `XDG_RUNTIME_DIR`. Video is unaffected.
- **Switching videos rapidly in some apps** can leave the transport controls
  on the phone out of step — a Play button while the video plays, or a broken
  scrubber. Tapping the button resyncs it. Apps that insert several queue
  items in quick succession (iQiyi) can also end up playing a different item
  than the one picked. Under investigation.

## 11. Reporting a problem

```sh
/opt/rplay/bin/rplay -n rPlay 2>&1 | tee /tmp/rplay.log
```

Include `/opt/rplay/bin/usb-host-services.sh status`, your Pi model and OS
version, the iPhone model and iOS version, and what you did and when.
