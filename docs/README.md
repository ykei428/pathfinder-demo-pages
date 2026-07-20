Engine: Godot 4.6.3.stable (stock editor from godotengine.org)

Spine Editor: 4.2 exports in repo until re-exported — see note below

Spine GDExtension: `project/bin/` (spine-godot; runtime **4.3** as of 2026-05 extension update)

**Spine export note:** After updating the GDExtension, headless load reported `Skeleton version 4.2.43 does not match runtime version 4.3`. Re-export player/enemy skeletons from **Spine 4.3** (or install a 4.2-runtime extension build) before expecting in-game Spine to work.

**Godot path (Windows):** `C:\Program Files\Godot\Godot_v4.6.3-stable_win64\Godot_v4.6.3-stable_win64.exe` — set `GODOT_EXE` for MCP/scripts (see `tools/godot/README.md`).

Pin for current PowerShell session:

```powershell
$env:GODOT_EXE = 'C:\Program Files\Godot\Godot_v4.6.3-stable_win64\Godot_v4.6.3-stable_win64.exe'
# or:
. .\mcp_local\pin-godot-env.ps1
```

Language: GDScript

Copilot workflow notes:
- See `docs/copilot_workflow.md` for reusable prompt patterns and chat shortcut commands.
- Chat shortcuts currently documented there include `start MCP` and `push now`.

Progression and procedural flow notes:
- See [docs/game_overview.md](docs/game_overview.md) for a short pitch and loop.
- See [docs/character_stats.md](docs/character_stats.md) and [docs/ai/deep-research-stats.md](docs/ai/deep-research-stats.md) for the v1 stat system.
- See [docs/architecture.md](docs/architecture.md) and [docs/godot_topdown_spec.md](docs/godot_topdown_spec.md) for floor generation, room state ownership, and run/profile persistence rules.

Audio intake custom note:
- New gameplay SFX should be staged in `project/assets/audio/_raw/`, then moved into `project/assets/audio/` once selected for use.
- Keep the paired `.import` file with the asset and update its `source_file` path after moving so Godot tracks the new location correctly.
- For short gameplay one-shots, keep `importer="wav"` with `compress/mode=2` unless a specific memory/perf pass requires different compression.

## Windows local demo (shareable zip)

Native **Windows x64** build — same gameplay as the editor, no browser/WebAssembly.

1. One-time: Godot **Export Templates** installed (Editor → Manage Export Templates, same version as editor).
2. `project/export_presets.cfg` is in git (Web, Web Mobile Demo, Windows Desktop Demo). If a preset is missing locally, pull latest or see `mcp_local/export-presets/`.
3. Build (reads version from `project.godot`, suffixes output files):
   ```powershell
   & ".\mcp_local\create-demo.bat"
   ```
   Or say **create demo** in chat.
4. Run: `dist\Pathfinder-v0.6.7\Pathfinder-v0.6.7.exe` (keep matching `.pck` and Spine DLL in the same folder).
5. Share: send `dist\Pathfinder-v0.6.7-windows.zip`; recipient unzips and runs the versioned `.exe` on Windows 10/11 x64.

Requirements for recipients: Windows 64-bit only (not macOS/Linux). No Godot install needed.

## Web Demo / GitHub Pages

This project can be published as a small web demo while keeping the source repo private.

1. In Godot, use the `Web` export preset and export to `docs/index.html`.
2. Confirm the local export created web files in `docs/` (at minimum `index.html` plus matching `.js`, `.wasm`, and `.pck` files).
3. Run `publish demo` in chat, or run `mcp_local/publish-demo.bat` manually.
4. The publish flow syncs web export artifacts to a separate public demo repo (required when this source repo is private on GitHub Free), commits, and pushes.
5. In the public demo repo, enable GitHub Pages:
	- Source: **GitHub Actions** (workflow `Deploy GitHub Pages`)
	- Do **not** use “Deploy from branch” — dual deploys cause 409 conflicts
6. After publish, confirm live build: open `https://ykei428.github.io/pathfinder-demo-pages/DEMO_VERSION.txt` — must match `project/project.godot` `config/version`.

Publish uses a **content-hashed `mainPack`** (e.g. `index-v0-7-27-9118889a.pck`) so browsers cannot keep a stale broken pack. **`index.pck` alone is not the loader URL.** See **`docs/web_demo_publish.md`** for the black-floor regression and safeguards.

Web threading note:
- This repo includes threaded and non-threaded Spine web binaries.
- Prefer the normal single-threaded Web export preset unless you explicitly need browser threading.

### Web mobile demo (touch controls)

Phone browsers (landscape): separate export with on-screen stick + combat buttons.

1. Preset **Web Mobile Demo** is in `project/export_presets.cfg` (`custom_features=mobile_demo`).
2. Run `publish demo mobile` in chat, or `mcp_local/publish-demo-mobile.bat`.
3. Play at `https://ykei428.github.io/pathfinder-demo-pages/mobile/` (desktop keyboard demo at `/` is unchanged).
4. F5 testing: set `touch/force_enable=true` in `project/project.godot` temporarily.

Quick workaround test (before export preset exists):
- Create a temporary `docs/index.html` file.
- Run: `mcp_local/publish-demo.bat -SkipExport -AllowMissingWebArtifacts -DemoRepoPath "<path-to-public-demo-repo>"`
- This validates the private-repo to public-pages sync flow first, then you can switch to normal full exports.



