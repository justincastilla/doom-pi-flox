# wads/

Drop IWAD files here. The Flox activation hook puts this directory first on
`DOOMWADPATH`, so any engine in the environment finds them by name.

| File | Where it comes from | How to play |
| --- | --- | --- |
| `freedoom1.wad`, `freedoom2.wad`, `freedm.wad` | Already in the environment (the `freedoom` package). Nothing to copy. | `flox activate -- doom` |
| `doom1.wad` | Shareware Episode 1, freely redistributable. `flox activate -- get-shareware-wad` | `DOOM_IWAD=doom1.wad flox activate -- doom` |
| `DOOM.WAD`, `DOOM2.WAD` | Your own purchased copy (Steam, GOG, the 1993 floppies). Copy them here. | `DOOM_IWAD=DOOM.WAD flox activate -- doom` |

`.wad` files in this directory are ignored by git. Commercial WADs must never
be committed or pushed to FloxHub.
