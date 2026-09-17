# Booth runbook: We Are Developers 2026

## The night before

- [ ] Pi boots into DOOM on its own (kiosk unit enabled). Reboot it twice to be sure.
- [ ] `flox activate -- smoke-test` passes on the Pi. Screenshot the output for the booth screen.
- [ ] The laptop you'll demo from has the repo cloned and `flox activate -- doom -window` works (macOS Apple Silicon uses the `aarch64-darwin` entries of the same lockfile).
- [ ] Pack: Pi, PSU, micro-HDMI cable, USB keyboard, USB mouse, a gamepad, Ethernet cable, spare flashed SD card.
- [ ] `git pull` on both machines; the Pi's SD card should have the packages cached from the hotel.
- [ ] Optional: `flox push` done, so `flox activate -r <you>/doom-pi` works on a stranger's laptop.

## The 60-second demo

1. Point at the Pi playing DOOM. "That's a Raspberry Pi. Nothing on it was apt-installed except git and Flox."
2. Open the laptop, `cat .flox/env/manifest.toml`. Three packages, ten lines that matter.
3. `cd doom-pi-flox` on the laptop (auto-activation, the prompt flips to `flox [doom-pi]`), then `doom -window`. "Same manifest, same lockfile, different CPU and OS."
4. If they're technical: `.flox/env/manifest.lock`, scroll to `aarch64-linux`. "That's the exact build the Pi is running. The arm64 CI job runs it too."
5. If they have Flox installed: `flox activate -r <you>/doom-pi -- doom`. No clone.
6. Hand them the keyboard.

Talking points that land:
- Doom renders in software at 35 fps like it's 1993; the Pi is bored. The hard part was never the game, it was the dependency chain, and that's what the lockfile removes.
- `/usr` on the Pi is untouched. Someone can `apt upgrade` the Pi mid-demo and nothing changes.
- It is the real thing: id's 1993 shareware Episode 1, byte-identical to the 1995 `doom19s.zip`, with id's own "PLEASE DISTRIBUTE!!!" banner as the license. Bring your own `DOOM.WAD` and you get all four episodes.
- If John Romero is at the booth: E1M1 is his map. Hand him the keyboard.
- Two engines in one environment: `DOOM_PORT=chocolate-doom` for 320x200 purism, Crispy for widescreen. Same WAD.

## If it breaks

| What you see | Do this |
| --- | --- |
| Pi shows a login prompt, no game | `sudo systemctl restart doom-kiosk`, then `journalctl -u doom-kiosk -n 50`. |
| Game exits immediately in a loop | Stop the unit, run `flox activate -- doom` by hand and read the error. Usually display or WAD path. |
| Someone quit and it's stuck on the menu | That's not stuck, they saved. `rm -rf ~/doom-pi-flox/.doom-home && sudo systemctl restart doom-kiosk` resets. |
| No sound | Booth halls are loud anyway. `SDL_AUDIODRIVER=alsa` in the unit, or `-nosound` and move on. |
| Wi-Fi is dead | Nothing needs it after the packages are cached. Don't `git pull` at the booth. |
| SD card corrupted | Spare card. Packages are cached on it too if you activated once. |
| A visitor wants to try their own WAD | `cp` it into `wads/` from a USB stick, `DOOM_IWAD=whatever.wad flox activate -- doom`. |

## Reset between visitors

```bash
sudo systemctl stop doom-kiosk
rm -rf ~/doom-pi-flox/.doom-home
sudo systemctl start doom-kiosk
```

## Afterwards

- Note in `docs/FEASIBILITY.md` which of the "needs a real Pi" items passed, and on which Pi model and OS version.
- If the GPU path worked (`DOOM_RENDER=gpu`), flip the default in `bin/doom`.
