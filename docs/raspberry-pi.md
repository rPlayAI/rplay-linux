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
- For **control** (driving the phone): Bluetooth, plus reachability of an
  MFi control-license server. See §5.

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

## 5. Controlling the phone

Mouse, keyboard and touch on the mirror window are forwarded to the phone
over Apple's iAP protocol. Two things must be in place.

### 5.1 Bluetooth pairing

Pair the iPhone from the Pi's Bluetooth settings. Check it:

```sh
bluetoothctl info <phone-mac>
```

You want `Paired: yes`, `Trusted: yes`, and the service
`00000000-deca-fade-deca-deafdecacafe` — Apple's iAP accessory UUID.

### 5.2 MFi authentication

iAP requires Apple MFi authentication, which rPlay fetches from a
control-license server. Without a reachable one the log says:

```
failed to connect to control license server
Failed to connect to license Server:, unable to control iOS
[bt_thread] mfiauth_fetch_auth_info -> -1
```

Mirroring still works; only control is lost. Point rPlay at your server in
`/etc/rplay/conf`:

```sh
TARPLAY_MFI_HOST=192.168.1.130
TARPLAY_MFI_PORT=9010
```

Check reachability from the Pi:

```sh
timeout 5 bash -c 'echo > /dev/tcp/<host>/9010' && echo reachable
```

Working looks like `mfiauth_fetch_auth_info -> 0`, then
`pair_bluetooth_device -> true`, then the touchscreen descriptor
registering.

### 5.3 Enable a pointer mode on the phone

For touches to register, iOS needs an Accessibility pointer:

**Settings → Accessibility → Zoom → ON** (AssistiveTouch also works).

rPlay shows a reminder dialog about this the first time control becomes
active. Set `RPLAY_IAP_TOUCH_NO_REMINDER=1` in `/etc/rplay/conf` once you know.

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
| `TARPLAY_MFI_HOST` / `_PORT` | MFi control-license server |
| `RPLAY_USB_MIRROR` | Start with the USB mirror enabled |
| `RPLAY_SW_RENDER` | `0` forces the accelerated renderer |
| `RPLAY_IAP_TOUCH_NO_REMINDER` | Suppress the Zoom reminder dialog |
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
- **Control does nothing** — see §5. Almost always MFi or the Zoom setting.

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
