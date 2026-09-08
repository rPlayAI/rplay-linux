# Troubleshooting

## The iPhone shows up as a folder, and rPlay sees nothing

Your desktop grabbed it first. An iPhone is three devices to Linux —
lockdown (usbmuxd), AFC, and a PTP camera — and the GVFS volume monitors
open the camera one on every plug, even with auto-mount off.

```sh
/opt/rplay/bin/usb-host-services.sh status
```

Anything listed under "processes holding a USB device fd" that is not
`rplay_wd` will block us. Clear them:

```sh
/opt/rplay/bin/usb-host-services.sh disable
```

Then unplug and replug the phone. `enable` puts everything back.

## Nothing happens when I plug the phone in

- Use a **data** cable. Charge-only cables never enumerate; check with
  `lsusb | grep -i apple`.
- Unlock the phone first — iOS only offers "Trust This Computer" while
  unlocked.
- If this is the first time trusting this machine, **replug after
  tapping Trust**. Pairing completes on one connection, mirroring starts
  on the next.

## The mirror window does not come back after unplugging

Known issue. Reconnection depends on the order two events arrive in, and
sometimes the mirror does not restart. Unplug and replug again, or
restart rPlay.

## The phone appears, then disconnects a few seconds later

A cable or port problem, not rPlay. `dmesg | tail` will show the device
arriving and going away. Try another cable and a directly-connected port
rather than a hub.

## Mirroring over USB is very laggy on a Pi

Expected — see [usb-mirroring.md](usb-mirroring.md). USB delivers 2.9×
the pixels of AirPlay and a Pi 4 cannot software-decode that at rate. Use
AirPlay on a Pi; USB mirroring is fine on an x86 PC.

## Mouse and keyboard do not control the phone

The mirror works but the phone ignores you, and the log says:

```
failed to connect to control license server
Failed to connect to license Server:, unable to control iOS
[bt_thread] mfiauth_fetch_auth_info -> -1
```

Driving the phone uses Apple's iAP protocol, which requires MFi
authentication. rPlay fetches that from a control-license server, and
without one the phone will not accept input — mirroring is unaffected.

If your licence server is on your own network, point rPlay at it in
`/etc/rplay/conf`:

```sh
TARPLAY_MFI_HOST=192.168.1.130
TARPLAY_MFI_PORT=9010
```

Check it is reachable from the machine running rPlay:

```sh
timeout 5 bash -c 'echo > /dev/tcp/<host>/9010' && echo reachable
```

When it works the log reads `mfiauth_fetch_auth_info -> 0`, followed by
`pair_bluetooth_device -> true` and the touchscreen descriptor
registering.

The phone must also be paired over Bluetooth first — pair it from your
desktop's Bluetooth settings, and check with
`bluetoothctl info <mac>` that it shows `Paired: yes` and advertises
`00000000-deca-fade-deca-deafdecacafe`, Apple's iAP service.

## Audio comes out of the wrong output

Set it explicitly in `/etc/rplay/conf`:

```sh
RPLAY_AUDIO_DEVICE=hw:0      # HDMI
RPLAY_AUDIO_DEVICE=plughw:2  # 3.5 mm jack
```

`aplay -l` lists the cards. Without this, ALSA's `default` defers to
PipeWire, which often picks the jack.

## The phone does not see rPlay on the network

- Click **Start** — rPlay does not advertise until you do.
- Both devices must be on the same subnet; check that `rplay-mdnsd` is
  running and owns port 5353 (`sudo ss -lunp | grep 5353`). If
  `avahi-daemon` has it, disable avahi.

## Video stutters or stalls when casting YouTube

Raise the segment-fetch timeout in `/etc/rplay/conf` — the default suits
a wired link and a slow Wi-Fi connection can exceed it mid-segment:

```sh
RPLAY_PROXY_IDLE_MS=2500
```

## Collecting a log for a bug report

```sh
/opt/rplay/bin/rplay -n rPlay 2>&1 | tee /tmp/rplay.log
```

Include what you plugged in and when, and the output of
`usb-host-services.sh status`.
