# Web demo publish — black floor safeguard

**Symptom:** Hosted demo shows character + HUD on a **black void**; editor F5 is fine.

**Not the cause:** missing or wrong Godot export templates (4.6.3 templates export successfully).

This doc records the **June 2026** regression (`v0.7.26`–`v0.7.27`) so we do not repeat it.

---

## Root cause (three layers)

### 1. Trim textures excluded from the web `.pck`

Floor/wall PNGs under `project/assets/textures/trims/` were imported as **VRAM / S3TC** (`vram_texture: true` in local `.import` files — gitignored).

Godot **web export drops** those GPU-compressed blobs. The pack shrinks from a healthy size (historically **~67 MB**, currently **~26–30 MB** with the leaner asset set) to a stub pack (**~7–12 MB**, historically ~19 MB). At runtime `load()` for trim albedos fails → empty room preview → black background. Spine cards still draw.

**Detection:** `index.pck` size **&lt; 20 MB** after export, or missing `trim_brick_a_*` `.ctex` path entries in the pack.

### 2. `godot --import` before export reverted trims

Publish runs `--import` so the export is fresh. That step **regenerated** trim `.import` files back to VRAM unless we patch them again **after** import.

**Rule:** never run a manual web export with `--import` alone; use `publish-demo.bat`, which runs the trim prep loop.

### 3. Browser cache of bare `index.pck`

HTML was cache-busted (`index.js?v=X.Y.Z`) but the pack URL stayed **`index.pck`**. After a bad ~19 MB deploy, browsers kept that file while loading new HTML expecting ~67 MB → same black-floor symptom even when GitHub Pages served a good pack.

**Fix:** publish sets `mainPack` to a **content-hashed** filename, e.g. `index-v0-7-27-9118889a.pck`. Each good export gets a new URL.

---

## Automated safeguards (publish script)

`mcp_local/publish-demo.bat` / `publish-demo.ps1` must keep these steps **in order**:

1. `Clear-WebExportArtifacts` — no stale `.pck` accepted.
2. `Prepare-TrimTexturesForWebExport` — patch trim `.import` → `--import` → re-patch if import reverted VRAM (max 3 attempts).
3. Web `--export-release`.
4. `Assert-WebExportPackHasTrimTextures` — **fail if `index.pck` &lt; 20 MB** or trim `.ctex` entries missing.
5. `Update-WebExportCacheBust` — hash-suffixed `mainPack` + `fileSizes` in `index.html`.
6. `Write-DemoVersionManifest` — `pack=` line matches `mainPack`.
7. Sync demo repo + hash verify + **one** git push.
8. `Assert-LiveWebDemoPackAvailable` — live manifest version + pack **≥ 20 MB**.

If any step fails, **do not push** a partial export to the demo repo.

---

## Operator checklist

### Before publish

- [ ] Use **`publish demo`** or `mcp_local/publish-demo.bat` — not a raw Godot Export dialog alone.
- [ ] `GODOT_EXE` matches `project/project.godot` `config/features` (4.6.3).
- [ ] **One publish at a time** — wait for GitHub Pages deploy to finish (~1–2 min) before republishing. Rapid pushes cause **409 deployment already in progress** emails.

### After publish

- [ ] `DEMO_VERSION.txt` version matches `config/version`.
- [ ] `pack=` is a hashed name (`index-v…-xxxxxxxx.pck`), not bare `index.pck` alone.
- [ ] Network tab: pack download **~26–30 MB**, not a stub ~7–12 MB.
- [ ] Hard-refresh (**Ctrl+Shift+R**) or incognito when verifying.

### Demo repo (pathfinder_demo_pages)

- **Pages source:** GitHub Actions workflow `Deploy GitHub Pages` only — not “Deploy from branch” (dual deploys → 409 conflicts).
- Workflow uses `concurrency: cancel-in-progress: false` and waits for any in-flight Pages deployment before deploying.
- **409 failure emails:** GitHub sends one per failed workflow run when a new push starts while the previous Pages deploy is still running (~1–2 min). The site may still update from the latest successful run. Avoid back-to-back publishes; publish script blocks pushes within **120s** of the last demo commit.

---

## Manual diagnosis

| Check | Good | Broken |
|-------|------|--------|
| `index.pck` / `mainPack` size | ≥ 20 MB (typically ~26–30 MB; older deploys ~67 MB) | ~7–12 MB (or missing trim `.ctex`) |
| Floor visible in browser | yes | black void |
| `DEMO_VERSION.txt` `pack=` | `index-v…-hash.pck` | missing hash or stale |
| Trim in pack | lossless / embedded PNG | VRAM `.s3tc.ctex` excluded |

Local quick check after export:

```powershell
(Get-Item .\docs\index.pck).Length   # expect >= 40000000
Select-String -Path .\docs\index.html -Pattern 'mainPack'
Get-Content .\docs\DEMO_VERSION.txt
```

---

## Related

- `docs/export_parity_checklist.md` — F5 vs export gates
- `.cursor/rules/pathfinder-web-demo-publish.mdc` — agent guardrails
- `mcp_local/godot-export-utils.ps1` — `Fix-TrimTextureImportsForWebExport`, `Assert-WebExportPackHasTrimTextures`, `Update-WebExportCacheBust`
- Live demo: https://ykei428.github.io/pathfinder-demo-pages/
