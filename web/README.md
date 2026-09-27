# `web/` — the web-optimized derivatives localgpt.world serves

These are **not** the pack under `models/`. `website-world/scripts/add-world.mjs`
in the localgpt repository produces them with `@gltf-transform/cli`: textures
down to 512 px, then weld, simplify and prune, staying core glTF so the viewer
and `localgpt-gen --world` load them with no decoders. Regenerating them needs
that tool at a pinned version, so they are stored rather than rebuilt at deploy.

They live here rather than in the localgpt repository because they are content:
30 MB of glTF that would sit in a code repository's history forever and grow
with every published world. Here they are LFS objects in the repository that
already holds the pack.

| Path | What |
|---|---|
| `web/worlds/assets/models/` | the shrunk GLBs the curated gallery worlds reference |
| `web/worlds/assets/music/` | the soundtracks the Verse worlds perform to |
| `web/worlds/posters/` | one rendered frame per world, by `scripts/posters.mjs` |

`website-world/scripts/assemble.mjs` copies them into the site tree before
`npm run check` or a deploy; `$LOCALGPT_WORLD_ASSETS` points at this checkout.
