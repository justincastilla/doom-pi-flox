# Raspberry Pi setup

From a blank microSD card to DOOM at boot. Written for a Raspberry Pi 4 or 5;
a Pi 3 works at a lower resolution.

## 1. Flash the OS

Use Raspberry Pi Imager and pick **Raspberry Pi OS (64-bit)**. Lite is enough
for a kiosk; the desktop version also works. **It must be the 64-bit image**:
Flox packages are built for `aarch64-linux`, and the 32-bit OS reports
`armv7l`.

In Imager's settings, set the hostname, a user (the docs assume `pi`), enable
SSH and put in the venue Wi-Fi. Or bring an Ethernet cable.

Boot it, then:

```bash
uname -m            # must print aarch64
sudo apt update && sudo apt install -y curl
```

## 2. Install Flox

```bash
curl -fsSL https://get.flox.dev | sh
flox --version
flox config --set disable_metrics true     # optional
```

## 3. Make the game environment

You don't clone this repo on the Pi. You make an environment and install two
packages from the catalog: a Doom engine of your choice, and `doom_share`.

```bash
mkdir ~/doom && cd ~/doom
flox init
flox search doom                                   # every port here builds for aarch64-linux
flox install <engine> justincastilla/doom_share    # a software-rendered, vanilla-compatible port is the safe pick
```

The first install downloads roughly 100 MB, so do it at the hotel, not at the
booth. After that the environment is offline and instant.

If `doom_share` is not visible to you (personal catalogs are private to
their owner), build it from this repo on the Pi instead:

```bash
git clone https://github.com/justincastilla/doom-pi-flox ~/doom-pi-flox
cd ~/doom-pi-flox && flox build          # -> result-doom_share/
cd ~/doom && flox install <engine>
# then use ~/doom-pi-flox/result-doom_share/bin/doom_share in place of doom_share below
```

## 4. Play

From the console (Lite, or Ctrl-Alt-F2 on the desktop image):

```bash
cd ~/doom
flox activate -- doom_share
```

The launcher notices there is no Wayland or X11 session and sets
`SDL_VIDEODRIVER=kmsdrm`, so the engine takes over the display directly. On a
Pi it also defaults to SDL's software scaler, which is plenty for Doom. To try
the GPU path instead:

```bash
DOOM_SHARE_RENDER=gpu flox activate -- doom_share
```

From the desktop, run the same command in a terminal; add `-window` if you
want it windowed.

Sound: on the desktop SDL finds PipeWire on its own. On the console, if it's
silent, force ALSA:

```bash
SDL_AUDIODRIVER=alsa flox activate -- doom_share
```

Escape opens the menu; quit from there. Keys: arrows/WASD, Ctrl fire, Space
use, Shift run.

## 5. Boot straight into the game (kiosk)

```bash
sudo cp ~/doom-pi-flox/systemd/doom-kiosk.service /etc/systemd/system/   # or download just that file
# fix User=, Group=, WorkingDirectory=, HOME and ExecStart if your user or path differ
sudo systemctl daemon-reload
sudo systemctl enable --now doom-kiosk
```

The unit takes over tty1 and runs `flox activate -- doom_share --kiosk`.
When a player quits, the game restarts two seconds later. Config and saves go
to `~/.local/state/doom_share-kiosk/`, so a reset between players is
`rm -rf` on that directory.

```bash
journalctl -u doom-kiosk -f          # watch it
sudo systemctl stop doom-kiosk       # get your console back
sudo systemctl disable doom-kiosk    # stop it starting at boot
```

If you use the desktop image with auto-login, the desktop and the kiosk unit
will fight over the display. Use Lite for the kiosk, or set the Pi to boot to
console (`sudo raspi-config` > System Options > Boot / Auto Login).

## 6. Optional extras

**Gamepad.** Most ports ship a `<engine>-setup` tool; run it from the
activated environment, enable the joystick and bind buttons. Under the kiosk
unit the engine's config lives under `~/.local/state/doom_share-kiosk/`.

**Another engine.** `flox install gzdoom` then `doom_share --engine gzdoom`.
GZDoom needs a working GPU path, which is the one thing untested on a Pi.

**Your own DOOM.WAD.** Registered copies (Steam, GOG) work with any engine:
put the file somewhere and run `<engine> -iwad /path/to/DOOM.WAD`. Never
commit or publish it.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `uname -m` says `armv7l` | 32-bit OS. Re-flash with the 64-bit image. |
| `flox install` says no package for this system | Same cause. `aarch64-linux` is required. |
| `doom_share: no Doom engine found on PATH` | Install one in the same environment (`flox search doom`), or point at it with `--engine NAME` if its command doesn't contain "doom". |
| Black screen, then back to the shell | Read the last lines: an EGL/GBM error means the GPU path failed. Make sure `DOOM_SHARE_RENDER=gpu` isn't set, and that your user is in the `video` and `render` groups. |
| `Could not initialize SDL video: kmsdrm not available` | A compositor already owns the display. Run from a real tty (Ctrl-Alt-F2), or on the desktop unset `SDL_VIDEODRIVER`. |
| No sound on the console | `SDL_AUDIODRIVER=alsa`, and check `aplay -l` shows the HDMI device. |
| Keyboard does nothing under systemd | The unit needs `StandardInput=tty` and `TTYPath=/dev/tty1` (both set) and the user in the `input` group. |
| First activation is slow | It's downloading. Do it at the hotel. |
