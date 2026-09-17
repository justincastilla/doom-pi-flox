# DOOM on a Raspberry Pi, with Flox

One manifest. One lockfile. DOOM on a Raspberry Pi, on the laptop next to it,
and in CI, from the same three commands.

```bash
curl -fsSL https://get.flox.dev | sh        # install Flox (Pi OS 64-bit, Debian, Ubuntu, macOS)
git clone https://github.com/justincastilla/doom-pi-flox && cd doom-pi-flox
flox activate -- doom                       # rip and tear
```

No apt dependency hunting, no compiling SDL on a Pi, no "works on my SD card".
The environment pins [Crispy Doom](https://github.com/fabiangreffrath/crispy-doom)
and [Chocolate Doom](https://www.chocolate-doom.org/) for `aarch64-linux`,
`x86_64-linux`, `aarch64-darwin` and `x86_64-darwin`, so the exact same
binaries come up on every machine. The game is id Software's original 1993
shareware release, `doom1.wad` v1.9: Episode 1, *Knee-Deep in the Dead*,
the real E1M1. It ships in `wads/` because id shipped it that way (the
installer banner reads "SHAREWARE VERSION, PLEASE DISTRIBUTE!!!"). The free
[Freedoom](https://freedoom.github.io/) IWADs are in the environment too.

Built for the Flox booth at We Are Developers 2026. If John Romero walks past,
hand him the keyboard.

## What's in the box

| Path | What it does |
| --- | --- |
| `.flox/env/manifest.toml` | The whole environment: two engines, Freedoom, defaults, activation hook. |
| `.flox/env/manifest.lock` | Resolved package set for all four platforms. Commit it; activation needs no catalog lookup. |
| `wads/doom1.wad` | id Software's shareware DOOM v1.9 (Episode 1). MD5 `f0cefca49926d00903cf57551d901abe`, verified by the smoke test. |
| `bin/doom` | Launcher. Picks the engine and IWAD from `DOOM_PORT` / `DOOM_IWAD`, sets up SDL for a bare console or a desktop. |
| `bin/doom-kiosk` | Booth loop: relaunches the game whenever a player quits. Config and saves live in `.doom-home/`. |
| `bin/smoke-test` | Verifies the shareware WAD's checksum, then runs a headless timedemo in both engines with both IWADs. CI runs it on x86_64 and arm64. |
| `bin/enable-auto-activate` | One-time: make `cd` into this directory activate the environment (Flox native auto-activation). |
| `systemd/doom-kiosk.service` | Boot the Pi straight into DOOM on tty1. |
| `docs/` | [Feasibility study](docs/FEASIBILITY.md), [Raspberry Pi setup](docs/RASPBERRY-PI-SETUP.md), [booth runbook](docs/BOOTH-RUNBOOK.md). |

## Everyday commands

```bash
flox activate -- doom                              # shareware E1, Crispy Doom, fullscreen on a console
flox activate -- doom -window                      # windowed, on a desktop
DOOM_PORT=chocolate-doom flox activate -- doom     # 1993 mode: 320x200, no frills
DOOM_IWAD=freedoom1.wad flox activate -- doom      # Freedoom Phase 1 (four full episodes)
DOOM_IWAD=freedoom2.wad flox activate -- doom      # Freedoom Phase 2 (Doom II style maps)
flox activate -- doom -warp 1 1 -skill 4           # jump straight to E1M1 on Ultra-Violence
flox activate -- doom-kiosk                        # booth mode
flox activate -- smoke-test                        # headless proof it all works
flox activate                                      # interactive shell with everything on PATH
```

Any flag after `doom` goes straight to the engine (`-fullscreen`, `-nomusic`,
`-record`, `-playdemo`, `-file mymaps.wad`, ...). `crispy-doom-setup` and
`chocolate-doom-setup` are also on PATH for key bindings and gamepads.

## Auto-activate: `cd` is the whole setup

Flox can activate the environment when you enter the directory and
deactivate it when you leave, so the commands above lose their
`flox activate --` prefix. Consent is per user and per machine (a repo can't
switch it on for you), so run this once:

```bash
./bin/enable-auto-activate     # installs the Flox prompt hook, allows this directory
```

Then, in a new shell:

```bash
cd doom-pi-flox     # prompt becomes  flox [doom-pi]
doom                # everything in bin/ is on PATH
cd ..               # deactivated again
```

By hand it's two steps: put `eval "$(flox activate -d ~)"` (or `-D` if you
are logged in to FloxHub) in your shell rc to install the prompt hook, and
run `flox activate allow` inside the repo. `flox activate deny` reverses it.

## Raspberry Pi in five minutes

Tested target: Raspberry Pi 4 or 5 running Raspberry Pi OS 64-bit (Lite is
enough). Full walkthrough in [docs/RASPBERRY-PI-SETUP.md](docs/RASPBERRY-PI-SETUP.md).

```bash
sudo apt update && sudo apt install -y git curl
curl -fsSL https://get.flox.dev | sh
git clone https://github.com/justincastilla/doom-pi-flox ~/doom-pi-flox
cd ~/doom-pi-flox
flox activate -- smoke-test     # first activation downloads ~150 MB of packages
flox activate -- doom           # from the console, or from the desktop
```

To make it boot into the game, install the systemd unit (edit `User=` and the
paths if your user isn't `pi`):

```bash
sudo cp systemd/doom-kiosk.service /etc/systemd/system/
sudo systemctl daemon-reload && sudo systemctl enable --now doom-kiosk
```

## Why Flox for this

- **The same environment on the Pi and on your Mac.** `aarch64-linux` and
  `aarch64-darwin` sit side by side in one lockfile. Prep and test the booth
  demo on a laptop, then activate the identical environment on the Pi.
- **No system packages touched.** Everything lives under `/nix/store`; the
  Pi's own SDL, Mesa and libc are untouched and `apt upgrade` can't break the demo.
- **Reproducible by construction.** `manifest.lock` pins exact package
  revisions. A second Pi built next month gets bit-for-bit the same binaries.
- **Shareable.** `flox push` publishes the environment to FloxHub, after which
  anyone can `flox activate -r <owner>/doom-pi` with no git clone at all.

## Licenses

Scripts and docs in this repo are MIT. Crispy Doom and Chocolate Doom are
GPL-2.0-or-later, Freedoom is BSD-3-Clause. `wads/doom1.wad` is the
unmodified shareware version of DOOM, which id Software released for free
distribution; it is not covered by the MIT license and stays unmodified.
Commercial `DOOM.WAD` / `DOOM2.WAD` files are yours to bring; they are
git-ignored and must never be committed or pushed to FloxHub.
