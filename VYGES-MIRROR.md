# Vyges mirror notes

This repository is the Vyges mirror of [IHP-GmbH/IHP-Open-PDK](https://github.com/IHP-GmbH/IHP-Open-PDK).
Upstream content is unmodified. Everything Vyges adds is listed here.

## The tracked branch is `dev`, not `main`

`dev` is the default branch here and the branch the sync job follows.

On **2026-09-01** upstream merged the SG13CMOS-5L overlay into this repository
(PR **#1124**, "Migrate ihp-sg13cmos5l", head `ab1510c`). Since then the `dev`
tree root carries

```
ihp-common/        ihp-sg13cmos5l/        ihp-sg13g2/
Makefile.sg13cmos5l                       Makefile.sg13g2
```

`main`'s root still has only `ihp-sg13g2/`.

**Why that decides the branch.** The overlay is not a self-contained tree — 528
symlinks live in it, and the overlay's point back into the base with relative
paths:

```
ihp-sg13cmos5l/libs.ref/sg13cmos5l_sram       -> ../../ihp-sg13g2/libs.ref/sg13g2_sram
ihp-sg13cmos5l/libs.tech/gnucap/cpp/Makefile  -> ../../../../ihp-sg13g2/libs.tech/gnucap/cpp/Makefile
```

They resolve only when base and overlay are siblings under one root. Two
separately pinned repositories could satisfy that by convention; one tree
satisfies it by construction — and a `$PDK_ROOT` that points at the wrong place
fails loudly instead of producing a PDK with holes in it. `dev` also carries the
`rhigh` corner fix `4b7d7422` (verified: `compare/dev...4b7d7422` is `ahead_by: 0`,
i.e. an ancestor).

⚠️ **`dev` is a commit pin, not a release.** It has no upstream version tags —
quote the commit. Tagged releases (`v0.3.0` and earlier) remain on `main`.

⚠️ **The tree has submodules** (`ihp-sg13g2/libs.tech/{digital,palace,openems,…}`).
Clone with `--recurse-submodules`, or the PDK is incomplete in ways that surface
much later. Upstream's `.gitmodules` also declares several of them twice, which
git reports as `multiple configurations found … Skipping second one!` — noisy,
harmless, and upstream's to fix.

## What Vyges adds

| path | what it is |
| --- | --- |
| `ihp_sg13g2.vyges-pdk.json` | PDK descriptor for the base, pinned to this tree |
| `ihp_sg13cmos5l.vyges-pdk.json` | PDK descriptor for the overlay, same pin — moved here from `vyges-tools/ihp-sg13cmos5l`, which upstream PR #1124 supersedes |
| `.github/workflows/upstream-sync.yml` | daily mirror of `upstream/dev` |
| `VYGES-MIRROR.md` | this file |

Both descriptors carry `"ref": "dev@ab1510c…"`. Re-pinning means moving that
sha in both, then re-measuring — never one without the other.

## `.github/workflows/` is pinned, deliberately

The sync job makes this directory byte-identical to what is already on
`origin/dev` after every sync, so its push never contains a workflow diff.

Two reasons, and the first is a hard constraint: **`GITHUB_TOKEN` may not
create, update or delete a workflow file**, so a sync that carried an upstream CI
change is rejected outright — that is what failed run `33553104962`. The second
is on the merits: IHP's DRC/LVS/lint/symlink regressions are *their* CI, and
running them on a mirror spends Actions minutes on results nobody reads.

⟹ Changing a workflow here is a human action with a workflow-scoped token. The
sync never does it silently, and a conflict fails the run red rather than
leaving the mirror stale behind a green badge.
