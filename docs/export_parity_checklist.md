# Export parity checklist (F5 vs demo)

Use before **create demo**, **publish demo**, **publish demo mobile**, or after changes to export presets, enemy atlas, Spine/rat animation, or GDExtension layout.

**Rule:** Headless Godot load and a good editor session do **not** prove an exported build matches F5.

---

## Automated (run every time)

```powershell
# Readiness: Godot version, export templates, rat atlas in export_presets
& ".\mcp_local\verify-export-parity.bat"

# Headless project load (scripts, autoloads, room boot)
& ".\mcp_local\run-godot-headless.ps1" -ProjectPath ".\project"

# Room 2D vs 3D grid math at tile_size 2.0 / 2.5 / 3.0 m (see docs/tile_size_verify.md)
& ".\mcp_local\run-verify-tile-size.bat"
```

**create demo** runs readiness + Windows startup smoke after export. **publish demo** runs readiness before web export, verifies `config/version` inside the `.pck`, syncs to the public demo repo with hash checks, and **fails** if git reports no changes (no silent stale publish).

**publish demo mobile** uses preset **Web Mobile Demo** (`custom_features=mobile_demo` for touch UI). It exports to `docs/mobile/`, syncs to `docs/mobile/` in the demo repo, and verifies `https://ykei428.github.io/pathfinder-demo-pages/mobile/DEMO_VERSION.txt` after push. Desktop web at `/` is unchanged.

After **publish demo**, open `https://ykei428.github.io/pathfinder-demo-pages/DEMO_VERSION.txt` — must match `project/project.godot` `config/version`. Live **`pack=`** must be a hashed name (`index-v…-xxxxxxxx.pck`) and download **≥ 20 MB** (~26–30 MB with trims). **`index.js?v=X.Y.Z` busts script cache only** — the pack URL is the hash in `mainPack`. Full black-floor safeguards: **`docs/web_demo_publish.md`**.

Optional smoke on an existing Windows build:

```powershell
& ".\mcp_local\verify-export-parity.bat" -RunWindowsSmoke
```

### Automated pass criteria

- [ ] `GODOT_EXPORT_READY` — Godot exe matches `project/project.godot` → `config/features` (4.6.x)
- [ ] `EXPORT_TEMPLATES_OK` — templates installed for that exact version
- [ ] Export presets include `res://data/enemy_rat_atlas_data.tres` and `res://assets/spine/enemy_rat.atlas` (Web + Web Mobile Demo + Windows demo)
- [ ] Headless load: no script parse errors; dungeon/start room boots
- [ ] Tile size parity script: `VERIFY_TILE_SIZE_OK`
- [ ] Windows smoke (when run): no blocking log lines (e.g. `[EnemyRat] No frame regions parsed`, GDExtension mismatch)

---

## When `enemy_rat.atlas` changed

Readiness prints a reminder when checks pass:

`EXPORT_ATLAS_REGEN_CMD: python mcp_local/generate-enemy-rat-atlas-data.py`

Run that after editing `project/assets/spine/enemy_rat.atlas` (readiness **fails** if `.tres` is older than the atlas).

```powershell
python mcp_local/generate-enemy-rat-atlas-data.py
```

Then re-run automated checks and manual rat fight below.

Export presets must list `res://data/animations_table.csv` (player locomotion table — parity script checks this).

---

## Manual (required for animation feel)

Automated smoke does not judge locomotion smoothness or combat timing.

1. **F5** — start a run, fight rats (walk, attack, hit reactions, block if applicable)
2. **create demo** — run `dist/Pathfinder-vX.Y.Z/Pathfinder.exe` (or project-named exe in that folder)
3. **Spot-check** — same rat clips as F5; no jitter, wrong walk cycle, or frozen frames
4. **Web** (if publishing) — hard-refresh hosted demo after upload
5. **Web mobile** (if publishing) — phone landscape at `/mobile/`: stick move, tap/hold attack, hold block, dodge, back dodge

### Manual pass criteria

- [ ] Rat walk/attack/damage clips match F5 (no export-only wrong `SpriteFrames`)
- [ ] Player Spine card visible; no missing viewport texture on card
- [ ] No new errors in demo stdout/stderr smoke logs (if generated)

---

## Related

- `.cursor/rules/pathfinder-export-parity.mdc`
- `.cursor/rules/pathfinder-web-demo-publish.mdc`
- `docs/web_demo_publish.md`
- `/verify-export-parity` command
- `docs/reviews/full_code_review_phased.md` — Phase 4
