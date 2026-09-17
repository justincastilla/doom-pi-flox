# Feasibility: DOOM on a Raspberry Pi, packaged with Flox

**Verdict: feasible, and most of it is already proven.** Every component
needed is in the Flox catalog with an `aarch64-linux` build, the environment
locks for all four platforms, and both engines run a full timedemo inside the
activated environment. The one thing that cannot be verified without hardware
is video and audio output on the Pi itself, and there are cheap fallbacks for
each failure mode. Budget one evening with a Pi before the conference.

Date of study: 2026-09-17, Flox 1.16.0.

## What was verified, and how

| Claim | Evidence |
| --- | --- |
| Flox installs on Raspberry Pi OS (64-bit) | Flox ships `flox-<ver>.aarch64-linux.deb`; `curl -fsSL https://get.flox.dev \| sh` selects it on Debian-family aarch64. Same installer verified on Ubuntu x86_64 in this study. |
| Doom engines exist for aarch64-linux | `flox show crispy-doom` lists `aarch64-linux` (7.1). Same for `chocolate-doom` (3.1.1), `prboom-plus`, `dsda-doom`, `woof-doom`, `gzdoom`. |
| The real game can be shipped | id's shareware `doom1.wad` v1.9 (from the original `doom19s.zip`, MD5 `f0cefca49926d00903cf57551d901abe`) is committed in `wads/`. Both engines run its DEMO1 (5026 gametics) in the environment and auto-detect it ahead of Freedoom. |
| A free fallback IWAD exists | `freedoom` 0.13.0 (BSD-3-Clause), aarch64-linux build, ships `freedoom1.wad`, `freedoom2.wad`, `freedm.wad` under `share/games/doom`. |
| The manifest locks for the Pi | `manifest.lock` contains 12 entries: 3 packages x 4 systems including `aarch64-linux`. Committed to this repo. |
| The game actually runs | `flox activate -- smoke-test` runs `-timedemo demo1` through Crispy Doom and Chocolate Doom with SDL's dummy drivers: 7117 gametics at ~586 fps on the x86_64 container. |
| SDL can drive a Pi with or without a desktop | The SDL in the environment (sdl2-compat over SDL 3.4) has `kmsdrm`, `wayland`, `x11`, `opengles2` and `software` video backends and `pipewire`, `pulseaudio` and `alsa` audio backends compiled in. |
| CI can prove the arm64 build | GitHub-hosted `ubuntu-24.04-arm` runners are standard runners, available in public and private repos. The workflow runs the smoke test on both x86_64 and arm64. |

## What still needs a real Pi

These are the things a container cannot tell you. Each has a fallback baked
into `bin/doom` or documented in the runbook.

1. **Display on a bare console (Pi OS Lite).** SDL's `kmsdrm` backend must
   open `/dev/dri/card*` with the environment's own Mesa/GBM (from the Nix
   store, not the Pi's). The kernel side (vc4/v3d DRM) is the Pi's and is
   fine. Risk: EGL context creation fails with the Nix Mesa. Fallback:
   `SDL_RENDER_DRIVER=software`, which `bin/doom` already sets by default on
   a Pi. Doom renders in software anyway; the GPU only scales the frame.
2. **Display on the Pi OS desktop (Wayland, labwc).** SDL's Wayland backend
   plus the same Mesa question. Fallback: same software renderer, or run
   from a tty (Ctrl-Alt-F2) with kmsdrm.
3. **Audio.** On the desktop, PipeWire is running and SDL will find it. Under
   the systemd kiosk unit there is no user session, so use
   `SDL_AUDIODRIVER=alsa` (commented in the unit). The Pi 4/5 HDMI audio path
   is plain ALSA and works. Worst case: `-nosound` and let the booth speakers
   play the soundtrack from a phone.
4. **Performance.** Chocolate Doom on a Pi 5 is reported at 1-2 % CPU;
   Crispy Doom at 640x400 software-scaled to 1080p is still a light load. A
   Pi 4 is fine; a Pi 3 works at lower resolution (`-geometry 640x480` or
   Chocolate Doom). Untested by me on hardware.
5. **Input.** Keyboard and mouse over USB need no setup. For a gamepad, run
   `crispy-doom-setup` once and bind the buttons; the config persists in
   `.doom-home/` under kiosk mode.

## Hardware test checklist (one evening)

- [ ] Flash Raspberry Pi OS 64-bit **Lite** (Bookworm or Trixie), boot, `sudo apt update`.
- [ ] `curl -fsSL https://get.flox.dev | sh`, then `flox --version`.
- [ ] Clone this repo, `flox activate -- smoke-test`. First activation pulls ~150 MB.
- [ ] `flox activate -- doom` from the console. Expect fullscreen kmsdrm.
- [ ] If black screen or EGL error: `DOOM_RENDER=gpu` off (default) vs on, note which works.
- [ ] Sound check over HDMI. If silent: `SDL_AUDIODRIVER=alsa flox activate -- doom`.
- [ ] Install `systemd/doom-kiosk.service`, reboot, confirm it boots into the game.
- [ ] Optional: flash the full desktop image and repeat `flox activate -- doom -window` under Wayland.
- [ ] Optional: `flox auth login && flox push` so the booth story is `flox activate -r <you>/doom-pi`.

## Why not just `apt install chocolate-doom`?

You can, and it works. The point of the demo is what Flox adds on top:

- The identical environment on the presenter's Mac (`aarch64-darwin`) and the
  Pi (`aarch64-linux`): rehearse on the laptop, ship the SD card.
- Nothing in `/usr` is touched; `apt upgrade` on the morning of the show cannot
  change the demo.
- A lockfile you can hand to someone who builds a second Pi in six months.
- `flox push` / `flox activate -r` for the "no clone, no install, one command"
  moment at the booth.

## Alternatives considered

| Option | Why not (for this demo) |
| --- | --- |
| RetroPie / Batocera image | Full emulation frontend; buries the Flox story. Great product, wrong demo. |
| `gzdoom` | In the catalog for aarch64-linux and works, but needs a GPU path (OpenGL/Vulkan) which is the one thing untested. Crispy Doom keeps the demo software-rendered and boring in the right way. |
| Building Chocolate Doom from source on the Pi | Exactly the yak-shave Flox exists to remove. Could be a `[build]` section demo later if you want to show custom packages. |
| Commercial DOOM.WAD | Fine to *play* from your own copy (drop it in `wads/`), never to redistribute. The shareware `doom1.wad` is the real Episode 1 and is free to distribute unmodified, so it is the default; Freedoom stays as the fully free fallback with more maps. |

## Sources

- Flox install docs (aarch64 .deb, installer script): https://flox.dev/docs/install-flox/install/
- Flox manifest reference: https://flox.dev/docs/reference/command-reference/manifest.toml/
- Chocolate Doom on Raspberry Pi (wiki): https://www.chocolate-doom.org/wiki/index.php/Raspberry_Pi
- DOOM on a Raspberry Pi 5 (CPU load numbers): https://picockpit.com/raspberry-pi/doom-on-a-raspberry-pi-5/
- Crispy Doom: https://github.com/fabiangreffrath/crispy-doom
- Freedoom: https://freedoom.github.io/
- GitHub arm64 standard runners in private repos: https://github.blog/changelog/2026-01-29-arm64-standard-runners-are-now-available-in-private-repositories/
