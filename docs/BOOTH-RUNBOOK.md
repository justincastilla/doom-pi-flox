# Booth runbook: We Are Developers 2026

## The night before

- [ ] Pi boots into DOOM on its own (kiosk unit enabled). Reboot it twice to be sure.
- [ ] `./ci/smoke-test` passes on the laptop; screenshot the output for the booth screen.
- [ ] `flox run -p justin-flox/doom_share -- doom -window` works on the laptop. Apple Silicon uses the `aarch64-darwin` build of the same package.
- [ ] The publish workflow has run for all three platforms and `flox search doom_share` shows it. If visitors should be able to run it themselves, it was published to an organization catalog.
- [ ] Pack: Pi, PSU, micro-HDMI cable, USB keyboard, USB mouse, a gamepad, Ethernet cable, spare flashed SD card.
- [ ] Both machines have the packages cached from the hotel. Don't install at the booth.

## The 60-second demo

1. Point at the Pi playing DOOM. "That's a Raspberry Pi. Nothing on it was apt-installed except curl and Flox."
2. On the laptop, in a terminal with nothing set up: `flox run -p justin-flox/doom_share -- doom`. One command. No clone, no install, no environment. The game comes up.
3. "Same package on the Pi, on this Mac, and in CI on an arm64 runner. It's the 1993 shareware WAD plus a launcher, packaged with Flox from this repo, and the engine rides along as a runtime dependency."
4. If they're technical: `cat .flox/env/manifest.toml` in the repo. A `[build]` section that copies one file and verifies its checksum, and that's the whole package. `flox publish` re-clones and rebuilds it from git, so what's in the catalog is what's in the commit.
5. Hand them the keyboard.

Talking points that land:
- Doom renders in software at 35 fps like it's 1993; the Pi is bored. The hard part was never the game, it was the dependency chain, and that's what the lockfile removes.
- It is the real thing: id's shareware Episode 1, byte-identical to the 1995 `doom19s.zip`, with id's own "PLEASE DISTRIBUTE!!!" banner as the license.
- The launcher doesn't care which engine: `doom --engine <any port from the catalog>` and the WAD doesn't change.
- If John Romero is at the booth: E1M1 is his map. Hand him the keyboard.

## If it breaks

| What you see | Do this |
| --- | --- |
| Pi shows a login prompt, no game | `sudo systemctl restart doom-kiosk`, then `journalctl -u doom-kiosk -n 50`. |
| Game exits immediately in a loop | Stop the unit, `cd ~/doom && flox activate -- doom` by hand and read the error. Usually display. |
| Someone quit and it's on the menu | `rm -rf ~/.local/state/doom_share-kiosk && sudo systemctl restart doom-kiosk` resets. |
| No sound | Booth halls are loud anyway. `SDL_AUDIODRIVER=alsa` in the unit, or `-nosound` and move on. |
| Wi-Fi is dead | Nothing needs it after the packages are cached. |
| SD card corrupted | Spare card. Packages are cached on it too if you activated once. |

## Afterwards

- Note in `docs/FEASIBILITY.md` which of the "needs a real Pi" items passed, and on which Pi model and OS version.
- If the GPU path worked (`DOOM_SHARE_RENDER=gpu`), flip the default in `bin/doom_share`.
