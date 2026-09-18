# doom_share

id Software's original 1993 shareware DOOM, packaged with Flox.

`doom_share` is a Flox package containing the unmodified shareware IWAD,
`doom1.wad` v1.9 (Episode 1, *Knee-Deep in the Dead*, the real E1M1), and a
launcher that plays it with whichever Doom engine you install next to it. It
is data, not an engine, and it has no opinion about which engine you use.

```bash
flox search doom                                   # pick any engine
flox install <engine> justincastilla/doom_share
doom_share
```

That works on a Raspberry Pi (aarch64-linux), a Linux laptop, an Apple
Silicon or Intel Mac, and in CI, from the same package. The WAD is in this
repo because id shipped it that way: the 1995 installer's banner reads
"SHAREWARE VERSION, PLEASE DISTRIBUTE!!!".

Built for the Flox booth at We Are Developers 2026. If John Romero walks
past, hand him the keyboard. E1M1 is his map.

## Using the package

```bash
doom_share                         # E1M1, whichever engine is found first
doom_share -window                 # windowed, on a desktop
doom_share -warp 1 3 -skill 4      # any engine flag passes straight through
doom_share --engine <name>         # choose the engine (or: DOOMPORT=<name>)
doom_share --kiosk                 # booth mode: restarts when a player quits
doom_share --help
```

Engine lookup, in order: `--engine` or `DOOMPORT`; an executable called
`doom` on PATH; otherwise the first executable on PATH whose name contains
"doom" (setup tools and dedicated servers are skipped). If your engine's
command doesn't contain "doom", name it with `--engine`. Every Doom port in
the Flox catalog builds for aarch64-linux; a software-rendered,
vanilla-compatible one is the safe choice for a Pi.

On a Linux console with no desktop (Raspberry Pi OS Lite, or a tty) the
launcher selects SDL's KMS/DRM video driver, and on a Raspberry Pi it
defaults to SDL's software scaler. `DOOM_SHARE_RENDER=gpu` tries the OpenGL
ES path instead. With the shareware WAD loaded, engines refuse `-file`
PWADs, as id asked in 1995.

## What's in the repo

| Path | What it is |
| --- | --- |
| `.flox/env/manifest.toml` | The package definition: a `[build.doom_share]` that copies the WAD and the launcher into `$out`, pure-sandboxed, and verifies the WAD's checksum. |
| `wads/doom1.wad` | id Software's shareware DOOM v1.9, extracted from the original `doom19s.zip` DEICE installer. MD5 `f0cefca49926d00903cf57551d901abe`. |
| `bin/doom_share` | The launcher installed as `bin/doom_share` in the package. |
| `ci/smoke-test` | Builds the package, checks the WAD, and plays the built-in demo in a throwaway environment with one engine from the catalog (the test's choice, overridable with `TEST_ENGINE`). CI runs it on x86_64 and arm64. |
| `systemd/doom-kiosk.service` | Boot a Raspberry Pi straight into the game. |
| `docs/` | [Raspberry Pi setup](docs/RASPBERRY-PI-SETUP.md), [booth runbook](docs/BOOTH-RUNBOOK.md), [feasibility study](docs/FEASIBILITY.md). |

## Building and publishing

```bash
git clone https://github.com/justincastilla/doom-pi-flox && cd doom-pi-flox
flox build                         # -> result-doom_share/
./ci/smoke-test                    # build + checksum + play the demo headlessly
```

To publish it to your FloxHub catalog (the tree must be clean and pushed;
Flox re-clones and rebuilds from git so the published package is exactly
what's committed):

```bash
flox auth login
flox publish doom_share
flox search doom_share             # <your-handle>/doom_share
```

Packages in a personal catalog are visible only to that user. Sharing them
with everyone at a booth means publishing from an organization catalog
(`flox publish -o <org>`), which is a Flox for Teams feature.

## Raspberry Pi

Full walkthrough in [docs/RASPBERRY-PI-SETUP.md](docs/RASPBERRY-PI-SETUP.md).
The short version, on Raspberry Pi OS 64-bit:

```bash
curl -fsSL https://get.flox.dev | sh
mkdir ~/doom && cd ~/doom && flox init
flox install <engine> justincastilla/doom_share
flox activate -- doom_share
```

## Licenses

The launcher, build definition and docs are MIT. `wads/doom1.wad` is the
unmodified shareware version of DOOM, which id Software released for free
distribution on the condition that it stays unmodified; it is not covered by
the MIT license. Engines carry their own licenses. Commercial `DOOM.WAD` /
`DOOM2.WAD` files must never be committed here or published.
