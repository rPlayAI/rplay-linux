# Installing rPlay

## Raspberry Pi / Debian / Ubuntu

```sh
sudo apt-get install -y ./rplay_<version>_arm64.deb
```

That installs:

| | |
|---|---|
| `/opt/rplay/bin/rplay` | the receiver |
| `/opt/rplay/bin/rplay_wd` | the USB-mirroring daemon (optional) |
| `/opt/rplay/bin/mdnsd` | Bonjour, Apple's own implementation |
| `/etc/rplay/conf` | settings, all optional |

Launch it from your applications menu, or run `/opt/rplay/bin/rplay`.

## The first time you click Start

rPlay asks for your password **once**, through your desktop's normal
authentication dialog.

This is not rPlay wanting root for itself. A stock Linux desktop grabs an
iPhone the moment it is plugged in — `usbmuxd` takes one interface, and
the GVFS volume monitors open the device to enumerate it, which is why
the phone often appears as a folder. While they hold it, rPlay cannot
open it at all.

Clearing them needs root, so rPlay offers to do it once and remembers
that it asked. You can undo it at any time: the **AirPlay** and **USB**
tabs both have a **Restore desktop iPhone handling** button that puts
everything back the way your distribution shipped it.

To do it yourself instead:

```sh
/opt/rplay/bin/usb-host-services.sh status     # what is in the way
/opt/rplay/bin/usb-host-services.sh disable    # clear it
/opt/rplay/bin/usb-host-services.sh enable     # put it back
```

## Bonjour

rPlay ships Apple's `mdnsd` and runs it as `rplay-mdnsd`. It and
`avahi-daemon` both want UDP port 5353, so the package disables avahi. If
you need avahi back:

```sh
sudo systemctl disable --now rplay-mdnsd
sudo systemctl enable  --now avahi-daemon
```
