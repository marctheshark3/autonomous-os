# CAD

Mechanical source files for Lamp.

Large CAD binaries (`*.stp`, `*.step`, `*.stl`, `*.f3d`, `*.f3z`) are tracked
via **Git LFS**. See the repo-root `.gitattributes` for the filter rules.

## Files

| Folder | Contents |
|--------|----------|
| `step/` | 17 kit STEP parts (Inventor surface `OPEN_SHELL`) plus `{part}-fixed.stp` closed solids |
| `stl/` | the same kit STLs plus `{part}-fixed.stl` tessellations |

Kit originals (`base.stl`, `base.stp`, …) are the published Inventor surfaces and stay untouched. Printable 1:1 solids are the `*-fixed` files: sew the kit **outer** (or each disjoint body) to a `MANIFOLD_SOLID_BREP`. See `GOAL-closed-solids.md`.

The servo carriers are CNC aluminium; the wood trim is decorative CNC; everything else prints.

## Uploading a new revision

1. Install Git LFS once per machine: `brew install git-lfs && git lfs install`.
2. Drop the file in `hardware/cad/`.
3. Commit and push as normal:

   ```bash
   git add hardware/cad/step/<part>.stp hardware/cad/stl/<part>.stl
   git commit -m "cad: bump <part>"
   git push
   ```

   Git LFS handles the upload to GitHub's LFS storage automatically.
4. Update the table above (file + date) and commit `hardware/cad/README.md`.

## Cloning

A fresh clone needs LFS too. After `git clone`, run:

```bash
git lfs install
git lfs pull
```

to fetch the actual binaries (otherwise the working tree gets LFS pointer
stubs instead of the real files).

## Changelog

- **v3.1-fixed** (2026-08-30) — `{part}-fixed.stp` / `{part}-fixed.stl` for all 16 printable kit parts: sew kit outer (or all disjoint bodies) to a closed `MANIFOLD_SOLID_BREP`. Envelope 1:1 with kit (0.5 mm). Kit originals unchanged. `light-cover-fixed.stp` is a closed solid; its STL tessellation still has the kit’s 35 boundary edges. Staged on fork branch `cad/lamp-closed-solids` only — no PR to `autonomous-ai/autonomous-os`.
- **v3** (2026-05-20) — initial STEP export (`lamp-v3.stp`, since replaced by the per-part files above; see `../cad-archive-v0/`).
