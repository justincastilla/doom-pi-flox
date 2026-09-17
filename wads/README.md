# wads/

Drop IWAD files here. The Flox activation hook puts this directory first on
`DOOMWADPATH`, so any engine in the environment finds them by name.

| File | Where it comes from | How to play |
| --- | --- | --- |
| `doom1.wad` (committed) | id Software's shareware DOOM v1.9, extracted from the original `doom19s.zip` DEICE installer. MD5 `f0cefca49926d00903cf57551d901abe`. Episode 1, nine maps. The default. | `flox activate -- doom` |
| `DOOM.WAD`, `DOOM2.WAD` | Your own purchased copy (Steam, GOG, the 1993 floppies). Copy them here. | `DOOM_IWAD=DOOM.WAD flox activate -- doom` |

Only `doom1.wad` is tracked by git; every other `.wad` here is ignored.
Commercial WADs must never be committed or pushed to FloxHub.

About the shareware WAD: id released it for free distribution (the 1995
installer's banner reads "SHAREWARE VERSION / PLEASE DISTRIBUTE!!!") on the
condition that it stays unmodified, and id asked that no custom levels be made
for it. The engines honour that: with `doom1.wad` loaded, `-file` PWADs are
refused. Use a registered IWAD for custom maps.
