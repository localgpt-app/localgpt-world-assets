# localgpt-world-assets

LocalGPT's world asset pack: the 3D models and starter music behind
[LocalGPT Verse](https://github.com/localgpt-app/localgpt) (formerly
Reverie), the song-world app in the `localgpt` workspace (`crates/verse`),
which builds a 3D world for every song. Everything here is public domain
under [CC0 1.0](LICENSE).

The assets are kept apart from the app's code because they're large and
change on their own schedule. The workspace's `cargo xtask fetch-assets`
copies them in from a checkout of this repository, so no app downloads
anything at runtime.

## Contents

| Path | What it is |
|------|------------|
| `models/<id>/<id>.glb` | One model from [Poly Haven](https://polyhaven.com), packed into the single file Verse loads |
| `models/manifest.json` | The list of models Verse reads: placement data and provenance |
| `music/` | Four original tracks, `music.json` (the track list Verse reads) and `NOTICE` |
| `fetch_polyhaven.py` | Downloads the original models from Poly Haven (gitignored) and rewrites the manifest |
| `normalize.py` | Packs each model into its `.glb`; `--sync` copies the packed set to a folder you name |
| `generate_music.py` | Synthesizes the four tracks |
| `check.py` | Checks that both manifests match the files in the repository; CI runs it on every change |

### Models

171 models across Verse's eight worlds, in three placement tiers: hero
landmarks, medium props and ground cover. Each model carries a semantic
**kind** (`rock`, `tree`, `lamp`, …) — a small stable vocabulary over the
variants, with the concrete variant resolved per world and rotated
so repeats differ. The four extended worlds layer their own accents on top of
their base world's set.

| World | Models | Examples |
|-------|-------:|----------|
| Ember Flats | 36 | boulders, cliffs, quiver trees, dry branches, desert flowers |
| Velvet Circuit | 44 | an iron gate, factory facade, street lamps, crates, electronics, a street rat |
| Tide Gardens | 32 | reef rocks, a wooden pier, shark and ray statues, shrubs, a ukulele |
| Glass Expanse | 40 | moon rocks, chandeliers, statues, brass vases, a chess set |
| Cinder Reach | 6 | a fire pit, a barrel stove, lanterns (plus the Ember Flats set) |
| Mirage Circuit | 4 | a covered car, a suitcase, a ladder (plus the Velvet Circuit set) |
| Abyss Terraces | 4 | a ship, treasure chests, a compass (plus the Tide Gardens set) |
| Dawn Expanse | 5 | a jacaranda tree, dandelions, a garden gnome (plus the Glass Expanse set) |

Each entry in `models/manifest.json` has:

- `id` and `name`: the Poly Haven asset id and a display name
- `file`: the path to the `.glb`, relative to `models/`
- `kind`: the semantic kind (`rock`, `tree`, `lamp`, …) — a small stable
  vocabulary over the variants; 19 kinds across the pack
- `tier`: `hero`, `medium` or `scatter`
- `mood` and `mood_id`: the world, as a position in Verse's list and as its
  stable id (0 ember-flats, 1 velvet-circuit, 2 tide-gardens,
  3 glass-expanse, 4 cinder-reach, 5 mirage-circuit, 6 abyss-terraces,
  7 dawn-expanse). If Verse's worlds change, update `MOOD_IDS` in
  `fetch_polyhaven.py` and regenerate.
- `scale` and `dims`: a per-model adjustment and the native size in metres.
  Verse rescales each model to its tier's size.
- `license`, `author` and `source`: provenance. Verse's Credits screen shows
  each model's author and license.

The repository holds only what ships: the manifest and the `.glb` files,
about 522 MB. The `.glb` and `.mp3` files are stored in
[Git LFS](https://git-lfs.com) (see `.gitattributes`), so a fresh clone needs
LFS installed to fetch their contents; without it you get pointer files, which
is all `check.py` needs, since it only checks that the files exist. The
original downloads (about 1 GB) are gitignored; `fetch_polyhaven.py` fetches
them again when a model needs re-packing, and each entry's `source` link
records where it came from.

### Music

| Track | Tempo |
|-------|------:|
| Amber Drift | 68 BPM |
| Tidewater | 60 BPM |
| Nightglass | 92 BPM |
| Emberfall | 112 BPM |

Original compositions synthesized from scratch by `generate_music.py` (pads,
sine bass, plucked bells, soft percussion and reverb), with no samples or
recordings. They give Verse something to play before you import your own
music.

## Using with LocalGPT Verse

Keep this repository beside your checkout of the `localgpt` workspace,
whose `crates/verse` is LocalGPT Verse:

```
localgpt/                 the workspace (LocalGPT Verse is crates/verse)
localgpt-world-assets/    this repository
```

- **Packaging.** `cargo xtask verse-bundle` assembles a portable Verse into
  `crates/verse/dist/`: the release binary with an `assets/` folder beside
  it — fonts, the manifest, the `.glb` files and the starter tracks, taken
  from `VERSE_ASSETS`, else the sibling checkout. The macOS `LocalGPT.app`
  (`cargo xtask bundle`) carries no pack — that is fetched once per machine,
  as below — and its four starter tracks come from
  `LOCALGPT_STARTER_MUSIC`, the sibling checkout, or the pack.
- **Development.** From the workspace, `cargo xtask fetch-assets` copies the
  manifest, the `.glb` files it names and the starter tracks from the
  sibling checkout (`LOCALGPT_WORLD_ASSETS_SRC` names another one) into the
  pack every LocalGPT app reads (`~/.local/share/localgpt/models/pack`, or
  `LOCALGPT_WORLD_ASSETS`), and links `crates/verse/assets/models` to it.
  Run `git lfs pull` here first; the command is safe to re-run after
  pulling new assets, copying nothing twice. It only copies files, so it
  needs neither Node.js nor the original downloads.

## Regenerating

You need Python 3, Node.js for `normalize.py` (it packs models with
[glTF-Transform](https://gltf-transform.dev) through `npx`), and
[LAME](https://lame.sourceforge.io) for `generate_music.py`.

```sh
python3 fetch_polyhaven.py   # download the originals and rewrite the manifest
python3 normalize.py         # pack new models, point the manifest at the .glb files
python3 generate_music.py    # re-render the tracks and music.json
```

The fetch downloads the originals of every model that isn't already on disk:
on a fresh clone, all 171 (about 1 GB). Run `normalize.py` after every fetch:
the fetch writes `.gltf` paths into the manifest, and normalizing points them
back at the `.glb` files.

`normalize.py` packs only the models that don't have a `.glb` yet; `--force`
re-packs all of them. glTF-Transform is pinned to the version that built the
committed files, so re-packing doesn't drift with new releases.

To add a model, pick a CC0 model on Poly Haven, add a
`(slug, name, kind, tier, mood, scale)` row to `PACK` in
`fetch_polyhaven.py` (reusing an existing `kind` when one fits, so the
vocabulary stays small), run both scripts, and commit the new `.glb`
with the updated manifest.

## License

Everything in this repository is dedicated to the public domain under
[CC0 1.0 Universal](LICENSE).

- **Models** are by the artists of [Poly Haven](https://polyhaven.com), which
  publishes all its assets under CC0. Credit isn't required, but Poly Haven
  appreciates it, and Verse's Credits screen lists every model.
- **Music and scripts:** to the extent possible under law, the author(s) have
  dedicated all copyright and related and neighboring rights to them to the
  public domain worldwide. They are distributed without any warranty.

Only add CC0 material. This repository has a single license, and Verse ships
these files in an open `assets/` folder beside its binary. Marketplace
"royalty-free" models (TurboSquid, CGTrader, Fab) are ruled out for a second
reason: their licenses forbid shipping in an extractable format.

## In exported worlds

LocalGPT Verse can export a track's world (`VERSE_EXPORT_WORLD=<dir>`; see
its README): `<track id>.world.json`, the web viewer's format, and
`<track id>.world.ron`, Gen's save format. Placed models are referenced as
`models/<file>` from `models/manifest.json`, relative to the world's
`assets/` directory, so a world published with its assets needs only the
`.glb` files it names copied there; the localgpt.world gallery's copies are
web-optimized derivatives of them, kept in this repository's `web/` (see
`web/README.md`). Everything here is CC0, so that redistribution is
unrestricted; the music's `NOTICE` travels with it.
