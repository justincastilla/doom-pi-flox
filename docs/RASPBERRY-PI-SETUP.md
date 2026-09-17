# Raspberry Pi setup

From a blank microSD card to DOOM at boot. Written for a Raspberry Pi 4 or 5;
a Pi 3 works with Chocolate Doom and a lower resolution.

## 1. Flash the OS

Use Raspberry Pi Imager and pick **Raspberry Pi OS (64-bit)**. Lite is enough
for a kiosk; the desktop version also works if you want to alt-tab to a
terminal during the demo. **It must be the 64-bit image**: Flox environments
are built for `aarch64-linux`, and the 32-bit OS reports `armv7l`.

In Imager's settings, set the hostname, a user (the docs assume `pi`), enable
SSH and put in the venue Wi-Fi. Or bring an Ethernet cable; conference Wi-Fi
is a lottery.

Boot it, then:

```bash
uname -m            # must print aarch64
sudo apt update && sudo apt install -y git curl
```

## 2. Install Flox

```bash
curl -fsSL https://get.flox.dev | sh
flox --version
```

The installer detects Debian-on-aarch64 and installs the `.deb`, which also
registers an apt source for future upgrades. If you prefer to see the package
first:

```bash
curl -LO https://downloads.flox.dev/by-env/stable/deb/flox-1.16.0.aarch64-linux.deb
sudo apt install ./flox-1.16.0.aarch64-linux.deb
```

Optional, keeps the console quiet:

```bash
flox config --set disable_metrics true
```

## 3. Get the environment

```bash
git clone https://github.com/justincastilla/doom-pi-flox ~/doom-pi-flox
cd ~/doom-pi-flox
flox activate -- smoke-test
```

The first activation downloads the pinned packages (roughly 150 MB, so a
couple of minutes on venue Wi-Fi; do this at the hotel). The lockfile is
committed, so no catalog resolution and no FloxHub login is needed. You should
see:

```
crispy-doom      OK   (timed 7117 gametics in ... realtics (... fps))
chocolate-doom   OK   (timed 7117 gametics in ... realtics (... fps))
smoke-test: all engines ran the demo. The environment is playable.
```

## 4. Play

From the console (Lite, or Ctrl-Alt-F2 on the desktop image):

```bash
flox activate -- doom
```

`bin/doom` notices there is no Wayland or X11 session and sets
`SDL_VIDEODRIVER=kmsdrm`, so the engine takes over the display directly. On a
Pi it also defaults to SDL's software scaler (`SDL_RENDER_DRIVER=software`),
which is plenty for Doom. To try the GPU path instead:

```bash
DOOM_RENDER=gpu flox activate -- doom
```

From the desktop, just run the same command in a terminal; add `-window` if
you want it windowed.

Sound: on the desktop SDL finds PipeWire on its own. On the console, if it's
silent, force ALSA:

```bash
SDL_AUDIODRIVER=alsa flox activate -- doom
```

Escape opens the menu; quit from there. Keys: arrows/WASD, Ctrl fire, Space
use, Shift run.

### Skip the `flox activate --` prefix

Once per user on the Pi:

```bash
./bin/enable-auto-activate
```

That installs the Flox prompt hook in `~/.bashrc` and allows this directory,
so every new shell that `cd`s into `~/doom-pi-flox` lands in the environment
with `doom`, `doom-kiosk` and `smoke-test` on PATH. Leaving the directory
deactivates it. The systemd kiosk unit does not rely on this; it calls
`flox activate` explicitly.

## 5. Boot straight into the game (kiosk)

```bash
sudo cp systemd/doom-kiosk.service /etc/systemd/system/
# if your user or path differ, fix User=, Group=, WorkingDirectory=, HOME and ExecStart in the copy
sudo systemctl daemon-reload
sudo systemctl enable --now doom-kiosk
```

The unit takes over tty1 and runs `flox activate -- doom-kiosk`. When a
player quits, the game restarts two seconds later. Config and savegames land in
`~/doom-pi-flox/.doom-home/` so a reset is `rm -rf .doom-home`.

```bash
journalctl -u doom-kiosk -f          # watch it
sudo systemctl stop doom-kiosk       # get your console back
sudo systemctl disable doom-kiosk    # stop it starting at boot
```

If you use the desktop image with auto-login, the desktop and the kiosk unit
will fight over the display. Either use Lite for the kiosk or set the Pi to
boot to console (`sudo raspi-config` > System Options > Boot / Auto Login).

## 6. Optional extras

**Shareware E1M1.** id's shareware `doom1.wad` is freely redistributable:

```bash
flox activate -- get-shareware-wad
DOOM_IWAD=doom1.wad flox activate -- doom
```

**Your own DOOM.WAD.** Copy it from Steam/GOG into `wads/`. It's git-ignored.

```bash
DOOM_IWAD=DOOM.WAD flox activate -- doom
```

**Gamepad.** `flox activate -- crispy-doom-setup`, enable the joystick and bind
buttons. Under the kiosk unit the config file is in `.doom-home/.local/share/crispy-doom/`.

**Publish to FloxHub** so the booth pitch is one command with no clone:

```bash
flox auth login
flox push
# on any machine:
flox activate -r <your-floxhub-handle>/doom-pi -- doom
```

Note that `flox push` publishes the manifest, not the `wads/` directory, so
commercial WADs never leave your machine.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `uname -m` says `armv7l` | 32-bit OS. Re-flash with the 64-bit image. |
| `flox activate` says no packages for this system | Same cause, or `options.systems` was edited. Must include `aarch64-linux`. |
| Black screen, then back to the shell | Look at the last lines: an EGL/GBM error means the GPU path failed. `bin/doom` already forces the software renderer on a Pi; make sure `DOOM_RENDER=gpu` isn't set. Also confirm your user is in the `video` and `render` groups. |
| `Could not initialize SDL video: kmsdrm not available` | A compositor already owns the display. Run from a real tty (Ctrl-Alt-F2), or on the desktop unset `SDL_VIDEODRIVER`. |
| No sound on the console | `SDL_AUDIODRIVER=alsa`, and check `aplay -l` shows the HDMI device. |
| Game runs but keyboard does nothing under systemd | The unit needs `StandardInput=tty` and `TTYPath=/dev/tty1` (both set) and the user in the `input` group. |
| First activation is slow | It's downloading. Do it at the hotel, not at the booth. Afterwards activation is instant and offline. |
