# Plan: 1:1 closed solids for remaining lamp kit parts

## Goal kind
code-change

## Staging
- Working kit binaries: `/home/whaleshark/Documents/lamp/{stls,step}/` (real Inventor files).
- Product tree: `/home/whaleshark/Documents/lamp/autonomous-os` → **fork** `github.com/marctheshark3/autonomous-os`, path `robots/lamp/hardware/cad/{stl,step}/`.
- Do **not** push to `upstream` `github.com/autonomous-ai/autonomous-os`.
- Do **not** open a pull request against Autonomous Labs’ original repo. Staging is the fork only.
- Kit originals (`base.stl`, `base.stp`, and the other 15 part files) stay unchanged. New files are `{part}-fixed.stp` + `{part}-fixed.stl`.
- CAD binaries go through Git LFS (`robots/lamp/hardware/**/*.{stl,stp}`).
- The fork clone is sparse (`robots/lamp` only) and currently holds **LFS pointer stubs** (~130 B) for kit CAD, not the real blobs. Fixed solids are real files we add beside them.

## Same method as the base
Kit STEP files are Inventor `SHELL_BASED_SURFACE_MODEL` / `OPEN_SHELL`, not `MANIFOLD_SOLID_BREP`. Nested inner/outer skins became the “wall” (base ~0.10 mm; swivels ~0.20 mm; arms ~0.5–1.0 mm).

Fix = **sew the kit outer (or each disjoint body) into a closed solid**. Not a CadQuery remake, not a convex hull, not a Minkowski grow.

`base` is already done: `step/base-fixed.stp` + `stls/base-fixed.stl` (envelope delta 0, 283.44 cm³, 1 manifold solid).

## Parts

| Priority | Part | Kit issue | Extract |
|----------|------|-----------|---------|
| done | `base` | 2 nested OPEN_SHELLs, 0.10 mm, slice empty layers | outer only |
| P0 | `swivel-part-part1` `swivel-part-part2` `swivel-part-part3` | 2 nested, ~0.20 mm, slice empty layers | outer only |
| P1 | `arm-1-part1` `arm-1-part2` `arm-2-part1` `arm-2-part2` `cap-servo` | 2 nested, ~0.5–1.0 mm walls | outer only |
| P2 | `base-cap` | 5 disjoint OPEN_SHELLs (plate + 4 feet) | **all 5**, never keep-largest |
| P3 | `button` `neck` `head-part2` `head-part3` | 1 closed shell, still surface STEP | that shell |
| P4 | `head-part1` (3 boundary edges), `light-cover` (35 boundary edges) | open mesh | sew/close if 1:1 envelope holds |
| skip | `lamp` | full assembly, not a part | — |

Nested vs disjoint: if Σ|shell volumes| ≫ |net volume|, keep the **outer** (largest bbox). If Σ|shell volumes| ≈ |net volume|, keep **every** component.

## Acceptance criteria
1. Each in-scope part has a new `{part}-fixed.stp` from the kit outer (or all disjoint bodies), not a sketch remake and not a convex hull. Envelope matches the kit STL within **0.5 mm per axis**.
2. That STEP is a closed solid B-rep (`MANIFOLD_SOLID_BREP` / `CLOSED_SHELL`, not kit `OPEN_SHELL`). It is **non-convex** when the kit outer is (part volume ≪ bbox prism and ≪ convex hull). `base-cap` may be a compound of 5 solids.
3. Tessellation `{part}-fixed.stl` opened in Bambu is 1:1 on size (`bambu-studio --info` within 0.5 mm) and not a filled convex blob. `manifold = yes`. Nested two-shell parts become **1 part**; `base-cap` keeps 5 bodies.
4. Kit originals under `stls/` + `step/` and the fork’s kit LFS pointers are not overwritten.
5. Fixed files are committed on the fork (feature branch) under `robots/lamp/hardware/cad/{step,stl}/` via LFS. `base-printable.stl` (CadQuery remake) is **not** the deliverable and is not committed as the 1:1 fix.

## Verification plan
1. gating: geometric compare vs kit STL — bbox deltas, signed volume > 0, hull volume > part × 1.02, part < 85% of bbox prism. `{SCRATCH}/compare-{part}.log`.
2. gating: STEP text contains `MANIFOLD_SOLID_BREP` and `CLOSED_SHELL`, not kit two-`OPEN_SHELL` only.
3. gating: `validate_stl.py` — positive volume, envelope inside 256³, expected component count (1, or 5 for `base-cap`). min-wall **0.05** (this is 1:1 sew, not 2.4 mm manufacturing walls). `{SCRATCH}/validate-{part}.log`.
4. gating: `bambu-studio --info` size within 0.5 mm of kit; manifold yes. `{SCRATCH}/bambu-{part}.log`. If AppImage instance-lock, capture that and keep the geometric compare.
5. evidence: pytest parametrize over every `{part}-fixed.stl` that exists.

## Non-goals
- 2.4 mm manufacturing walls, Minkowski grow, Rage Industries branding, new grille patterns
- CadQuery sketch remakes as the 1:1 deliverable
- Sending a print job
- Replacing `lamp.stl` / `lamp.stp` assembly
- Force-push, commits, or a pull request to `autonomous-ai/autonomous-os` (Autonomous Labs original)

## Task checklist
- [x] Sew kit `base` outer to closed STEP + tessellation; kit files untouched
- [x] Write this goal and part inventory
- [x] P0 swivels
- [x] P1 arms + cap-servo
- [x] P2 base-cap (keep 5 bodies)
- [x] P3 single-shell button/neck/heads
- [x] P4 `head-part1` closed; `light-cover` STEP closed, STL tessellation still open (35 boundary edges)
- [ ] Push `*-fixed` to `marctheshark3/autonomous-os` only (LFS, feature branch; **no PR** to Autonomous Labs)
- [ ] Compare + validate_stl + bambu --info + pytest

## Risks
- Sewing the outer **fills the inner cavity** (same as base: solid of the outer envelope). That is the 1:1 sew, not a fused hollow wall.
- `base-cap` keep-largest would drop the four feet.
- Open `light-cover` / `head-part1` may not sew closed; then report HARD and leave kit files as-is.
- Bambu AppImage often cannot start a second CLI while the GUI is open.
- Fork CAD kit files are LFS stubs until `git lfs pull`; do not treat stub size as geometry.
