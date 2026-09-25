# doom_share

id Software's original 1993 shareware DOOM, packaged with Flox.

`doom_share` is a Flox package containing the unmodified shareware IWAD,
`doom1.wad` v1.9 (Episode 1, *Knee-Deep in the Dead*, the real E1M1), a
launcher installed as `doom`, and one Doom engine as a runtime dependency so
the package runs on its own.

```bash
flox run doom
```

That's it. No environment, no install step. It works on a Raspberry Pi
(aarch64-linux), a Linux laptop and an Apple Silicon Mac from the same
package. The WAD is in this repo because id shipped it that way: the 1995
installer's banner reads "SHAREWARE VERSION, PLEASE DISTRIBUTE!!!".

If more than one package in the catalog provides a `doom` command, name it:

```bash
flox run -p justincastilla/doom_share -- doom
```

Or install it into an environment and run `doom` like any other command:

```bash
flox install justincastilla/doom_share
doom
```

Built for the Flox booth at We Are Developers 2026. If John Romero walks
past, hand him the keyboard. E1M1 is his map.

## Using the package

```bash
doom                               # E1M1
doom -window                       # windowed, on a desktop
doom -warp 1 3 -skill 4            # any engine flag passes straight through
doom --engine <name>               # use another engine on PATH (or: DOOMPORT=<name>)
doom --kiosk                       # booth mode: restarts when a player quits
doom --help
flox run -p justincastilla/doom_share -- doom --kiosk     # the same through flox run
```

`doom_share` is an alias for `doom` inside the package.

**Engines.** The launcher names no engine. It uses `--engine` or `DOOMPORT`
if set, otherwise the first Doom engine it finds on PATH (an executable
called `doom` that isn't itself, then anything whose name contains "doom",
skipping setup tools and dedicated servers). The package bundles one engine
as a runtime dependency so `flox run doom` works with nothing else
installed; which one is a single line in the manifest's `[install]` and
`runtime-packages`, and any other port from the catalog works via
`--engine`.

On a Linux console with no desktop (Raspberry Pi OS Lite, or a tty) the
launcher selects SDL's KMS/DRM video driver, and on a Raspberry Pi it
defaults to SDL's software scaler. `DOOM_SHARE_RENDER=gpu` tries the OpenGL
ES path instead. With the shareware WAD loaded, engines refuse `-file`
PWADs, as id asked in 1995.

## What's in the repo

| Path | What it is |
| --- | --- |
| `.flox/env/manifest.toml` | The package definition: a `[build.doom_share]` that copies the WAD and the launcher into `$out`, pure-sandboxed, verifies the WAD's checksum, and carries one engine as `runtime-packages`. |
| `wads/doom1.wad` | id Software's shareware DOOM v1.9, extracted from the original `doom19s.zip` DEICE installer. MD5 `f0cefca49926d00903cf57551d901abe`. |
| `bin/doom_share` | The launcher, installed in the package as `bin/doom` (with `bin/doom_share` as an alias). |
| `ci/smoke-test` | Builds the package, checks the WAD, plays the built-in demo with `bin/doom` and nothing else installed, then again with another engine via `--engine`. CI runs it on x86_64 and arm64. |
| `.github/workflows/publish.yml` | Publishes to FloxHub from x86_64, arm64 and Apple Silicon runners. Needs a `FLOXHUB_TOKEN` secret; takes an optional organization. |
| `systemd/doom-kiosk.service` | Boot a Raspberry Pi straight into the game. |
| `docs/` | [Raspberry Pi setup](docs/RASPBERRY-PI-SETUP.md), [booth runbook](docs/BOOTH-RUNBOOK.md), [feasibility study](docs/FEASIBILITY.md). |

## Building and publishing

```bash
git clone https://github.com/justincastilla/doom-pi-flox && cd doom-pi-flox
flox build                         # -> result-doom_share/
./ci/smoke-test                    # build + checksum + play the demo headlessly
```

### Publishing

A package is published per platform, from a machine of that platform. The
`publish` workflow does all three at once (x86_64-linux, aarch64-linux for
the Pi, aarch64-darwin for Apple Silicon):

1. On a machine where you're logged in, `flox auth token` prints your token.
   Add it to the repo as the Actions secret `FLOXHUB_TOKEN`.
2. Actions tab, **publish**, Run workflow. Leave `org` empty for your
   personal catalog, or give an organization name.

By hand it's the same three commands, once per platform, with a clean and
pushed tree (Flox re-clones and rebuilds from git, so what's published is
exactly what's committed):

```bash
flox auth login
flox build doom_share
flox publish doom_share            # or: flox publish -o <org> doom_share
```

**Who can see it.** A personal catalog is visible only to its owner, so
`flox run doom` works for you but not for a booth visitor. Publishing to an
organization catalog makes it available to that organization's members; that
is a Flox for Teams feature. There is no fully public catalog for
user-published packages today, so "anyone in the world can `flox run doom`"
isn't something this repo can grant on its own.

## Raspberry Pi

Full walkthrough in [docs/RASPBERRY-PI-SETUP.md](docs/RASPBERRY-PI-SETUP.md).
The short version, on Raspberry Pi OS 64-bit:

```bash
curl -fsSL https://get.flox.dev | sh
flox run doom
```

## Licenses

The launcher, build definition and docs are MIT. `wads/doom1.wad` is the
unmodified shareware version of DOOM, which id Software released for free
distribution on the condition that it stays unmodified; it is not covered by
the MIT license. The bundled engine and any other you use carry their own
licenses. Commercial `DOOM.WAD` /
`DOOM2.WAD` files must never be committed here or published.
