# `web/` — the web-optimized derivatives localgpt.world serves

These are **not** the pack under `models/`. `cargo xtask add-world` in the
localgpt repository produces them with `@gltf-transform/cli`: textures down to
512 px, then weld, simplify and prune, staying core glTF so the viewer and
`localgpt-gen --world` load them with no decoders. Regenerating them needs that
tool at a pinned version, so they are stored rather than rebuilt at deploy.

Each gallery world is a `.world` package on the site, and its assets are kept
here the way the package holds them: by content hash, `<sha256>.<ext>`, under
`web/worlds/<name>.world/assets/`. The package's `manifest.json` and
`package.json` are tracked in the localgpt repository (`website-world/worlds/`),
so a world's text and its binaries meet only at assemble time.

They live here rather than in the localgpt repository because they are content:
30 MB of glTF that would sit in a code repository's history forever and grow
with every published world. Here they are LFS objects in the repository that
already holds the pack.

| Path | What |
|---|---|
| `web/worlds/<name>.world/assets/` | the package's shrunk GLBs and its soundtrack, named by their SHA-256 as the manifest references them |
| `web/worlds/posters/` | one rendered frame per world, by `scripts/posters.mjs` |

`cargo xtask assemble world --all` copies them into the site tree before
`npm run check:all` or a deploy; `$LOCALGPT_WORLD_ASSETS` points at this checkout.
