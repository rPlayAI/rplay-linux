# Running without a desktop, or on boot

`rplay` waits for you to click **Start** by default. For an appliance or
an init script, three flags remove the need:

| flag | effect |
|---|---|
| `--autostart`, `-a` | start the receiver at launch |
| `--usb-mirror` | also bring up the USB mirror (implies `--autostart`) |
| `--no-host-prepare` | skip clearing usbmuxd / the GVFS volume monitors |

`--no-host-prepare` matters unattended: clearing those services needs
root, and an unattended run cannot answer a password dialog. Prepare the
host once by hand instead:

```sh
sudo /opt/rplay/bin/usb-host-services.sh disable
```

## systemd unit

rPlay needs a display, so it belongs to a graphical session, not
`multi-user.target`:

```ini
# /etc/systemd/system/rplay.service
[Unit]
Description=rPlay AirPlay receiver
After=graphical.target rplay-mdnsd.service
Wants=rplay-mdnsd.service

[Service]
User=pi
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/pi/.Xauthority
Environment=XDG_RUNTIME_DIR=/run/user/1000
EnvironmentFile=-/etc/rplay/conf
ExecStart=/opt/rplay/bin/rplay -n rPlay --autostart --no-host-prepare
Restart=on-failure

[Install]
WantedBy=graphical.target
```

```sh
sudo systemctl enable --now rplay
journalctl -u rplay -f
```

Adjust `User`, `XDG_RUNTIME_DIR` (it is `/run/user/<uid>`) and the
`XAUTHORITY` path for your login manager — under GDM it is usually
`/run/user/<uid>/gdm/Xauthority`.

## Port 7000

AirPlay listens on TCP 7000. That is a privileged port on some systems;
check with:

```sh
cat /proc/sys/net/ipv4/ip_unprivileged_port_start
```

If it is above 7000, either lower it, grant the capability with
`sudo setcap cap_net_bind_service=+ep /opt/rplay/bin/rplay`, or run as
root.
