# doom

id Software's original 1993 shareware DOOM, packaged with Flox.

The unmodified shareware IWAD, `doom1.wad` v1.9 (Episode 1, *Knee-Deep in the
Dead*, the real E1M1), ships as two packages built from this repo.

**`doom`** is the game and an engine together. There is nothing to install:

```bash
flox run -p flox-labs/doom -- doom
```

**`doom_share`** is the game data on its own, with a launcher that plays it
with whichever Doom engine you install next to it. It is data, not an engine,
and it has no opinion about which engine you use:

```bash
flox search doom                                   # pick any engine
flox install <engine> justincastilla/doom_share
doom_share
```

Both work on a Raspberry Pi (aarch64-linux), a Linux laptop, an Apple
Silicon Mac, and in CI, from the same repo. The WAD is in this
repo because id shipped it that way: the 1995 installer's banner reads
"SHAREWARE VERSION, PLEASE DISTRIBUTE!!!".

The two packages provide the same files under `share/`, so install one or the
other into an environment rather than both.

Built for the Flox booth at We Are Developers 2026. If John Romero walks
past, hand him the keyboard. E1M1 is his map.

## Using `doom`

```bash
doom                               # E1M1
doom -window                       # windowed, on a desktop
doom -warp 1 3 -skill 4            # any engine flag passes straight through
doom --engine <name>               # use another engine (or: DOOMPORT=<name>)
doom --kiosk                       # booth mode: restarts when a player quits
doom --help
```

The engine, Crispy Doom, is a runtime dependency of the package, so it is
already on PATH and needs no environment of its own. `--engine` is there for
pointing at a different port you have installed.

## Using `doom_share`

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
| `.flox/env/manifest.toml` | Both package definitions. `[build.doom_share]` copies the WAD and the launcher into `$out`, pure-sandboxed, and verifies the WAD's checksum; `[build.doom]` takes that build's `share/` through `${doom_share}` and adds Crispy Doom as a runtime package. |
| `wads/doom1.wad` | id Software's shareware DOOM v1.9, extracted from the original `doom19s.zip` DEICE installer. MD5 `f0cefca49926d00903cf57551d901abe`. |
| `bin/doom_share` | The launcher installed as `bin/doom_share` in the `doom_share` package. |
| `bin/doom` | The launcher installed as `bin/doom` in the `doom` package. It needs no engine discovery, so it is the shorter of the two. |
| `ci/smoke-test` | Builds both packages, checks the WAD, plays the built-in demo in a throwaway environment with one engine from the catalog (the test's choice, overridable with `TEST_ENGINE`), and plays it again through `doom` with nothing on PATH. CI runs it on x86_64 and arm64. |
| `systemd/doom-kiosk.service` | Boot a Raspberry Pi straight into the game. |
| `docs/` | [Raspberry Pi setup](docs/RASPBERRY-PI-SETUP.md), [booth runbook](docs/BOOTH-RUNBOOK.md), [feasibility study](docs/FEASIBILITY.md). |

## Building and publishing

```bash
git clone https://github.com/justincastilla/doom-pi-flox && cd doom-pi-flox
flox build                         # -> result-doom_share/ and result-doom/
./ci/smoke-test                    # build + checksum + play the demo headlessly
```

To publish to your FloxHub catalog (the tree must be clean and pushed; Flox
re-clones and rebuilds from git so the published package is exactly what's
committed):

```bash
flox auth login
flox publish doom
flox search doom                   # <your-handle>/doom
```

`flox build` targets the machine it runs on, so covering all three systems
means publishing the same version once from each: x86_64-linux,
aarch64-linux and aarch64-darwin. They land as a single catalog entry. The
build job in `.github/workflows/smoke-test.yml` runs on all three and carries
the publish step commented out.

Packages in a personal catalog are visible only to that user. Sharing them
with everyone at a booth means publishing from an organization catalog
(`flox publish -o <org>`), which is a Flox for Teams feature.

## Raspberry Pi

Full walkthrough in [docs/RASPBERRY-PI-SETUP.md](docs/RASPBERRY-PI-SETUP.md).
The short version, on Raspberry Pi OS 64-bit:

```bash
curl -fsSL https://get.flox.dev | sh
flox run -p flox-labs/doom -- doom
```

Or with an engine of your own choosing:

```bash
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
