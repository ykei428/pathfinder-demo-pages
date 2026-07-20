# 3D presentation stack

How Pathfinder draws **2D gameplay** with **3D trim sheets** and **billboard entity cards**. Read this before changing draw order, shadows, or character shaders.

**Related:** [architecture.md](architecture.md) · [godot_topdown_spec.md](godot_topdown_spec.md) · [combat_hitbox_tuning.md](combat_hitbox_tuning.md) · [ai/flow_debug.md](ai/flow_debug.md)

---

## Render priority ladder

| Layer | `render_priority` | Depth | Notes |
|-------|-------------------|-------|--------|
| Ground/blob fake shadows | −18 … −16 | `depth_test_disabled` | `sh_shared_fake_shadow_3d.gdshader` |
| Trim walls **behind** player | `Game.WALL_BEHIND_RENDER_PRIORITY` (0) | opaque write + test | Per-edge `WallMultiMesh_*` |
| Floor AO overlay | 1 | `depth_draw_never` | |
| **Loot pickups** (placeholder spheres) | `Game.LOOT_RENDER_PRIORITY` (7) | `no_depth_test` on material | Behind player/enemy cards |
| **Entity cards** (player + enemies) | `Game.CARD_RENDER_PRIORITY` (8) | `depth_draw_never`, **`depth_test_disabled`** | Y-sorted via `sorting_offset` |
| Trim walls **in front** of player | `Game.WALL_FRONT_RENDER_PRIORITY` (9) | opaque write + test | Occludes cards when past room edge |
| Fog of war | 10 | `depth_test_disabled` | |
| FX decals / shockwaves | up to 127 | usually `depth_test_disabled` | |

Constants live in [`project/autoload/Game.gd`](../project/autoload/Game.gd).

---

## Entity cards (player + enemies)

- **Player:** `SubViewport` sized by **Graphics quality** (512 / 768 / 1024) → `Sprite3D` + [`sh_surface_player_spine_3d.gdshader`](../project/assets/materials/shaders/surface/sh_surface_player_spine_3d.gdshader). Default Medium = 768².
- **Enemies:** `AnimatedSprite2D` frame copied to `Sprite3D`; `StandardMaterial3D` with `no_depth_test` for wall sort parity.
- **Y-sort among cards:** `sorting_offset = Game.card_sorting_offset(global_position.y)` — south (larger gameplay Y) draws in front. Updated in `Player._sync_player_3d_projection()` and `EnemyAgent._sync_3d_card_transform()`.
- **Do not** use 2D sprite fallback for draw order (architecture invariant).

---

## Lighting & anti-aliasing policy

**Authority:** Workbench → published `RoomConfig` → `RoomRenderApply` (lights + Environment). Player **Graphics quality** (Settings) overlays AA, dir shadows, Spine viewport size, and optional glow.

| Use | Reject |
|-----|--------|
| Baked AO in trim ORM + floor AO overlay | Default **SSAO / SSIL / SDFGI** (fights cards + backface ghost; costly on web) |
| Directional fill + Spot key + optional Omni torch | Many shadow-casting Omnis |
| Fake card shadows (`sh_shared_fake_shadow_3d`) | Engine shadow maps on Spine/enemy cards |
| MSAA 2x/4x and/or FXAA via quality preset | TAA as default (ghosting on 2.5D cards) |
| Soft Environment glow on **High** only | Volumetric fog (conflicts with fog-of-war plane) |
| Spot light cookie on key light | Volumetric light shafts (requires engine volumetric fog) |

**Quality tiers** ([`Settings.gd`](../project/autoload/Settings.gd)):

| Preset | AA | Dir shadows | Spine VP | Glow |
|--------|-----|-------------|----------|------|
| Low | FXAA | Off | 512 | Off |
| Medium (default) | MSAA 2x | Off | 768 | Off |
| High | MSAA 4x | Soft on trim | 1024 | Soft |

Web clamps expensive bits (no MSAA 4x / no dir shadows / no glow). Trim MultiMeshes cast shadows only when dir shadows are active.

**F5/F4 baseline (before changing AA/shadows/SubViewport):** character room + combat room — idle 5s → walk → light attacks → charge release; compare `frame_ms` / `draw_calls` / spikes. See [pathfinder-perf-debug](../.cursor/rules/pathfinder-perf-debug.mdc).

---

## Trim walls vs cards

Walls are thin planes on room edges ([`RoomPreviewBuild.collect_wall_transforms_by_edge`](../project/scripts/RoomPreviewBuild.gd)). Four `MultiMeshInstance3D` nodes (`WallMultiMesh_North` … `West`) each get a **duplicated** material so priority and backface can differ per edge.

[`WallDrawOrder3d`](../project/scripts/WallDrawOrder3d.gd) (child of `LevelRoot`) each frame:

1. Reads player `global_position` vs [`RoomConfig.gameplay_wall_bounds_global`](../project/scripts/RoomConfig.gd).
2. Sets edge material `render_priority` to **0** (behind) or **9** (in front):
   - South in front when `player.y > south_y`
   - North in front when `player.y < north_y`
   - East in front when `player.x > east_x`
   - West in front when `player.x < west_x`
3. **South backface gate:** removes `next_pass` (`sh_wall_backface`) when player is within `south_backface_suppress_margin_tiles` of the south wall (reduces interior “ghost” overlay).

Cards use `depth_test_disabled` so they paint over **behind** walls without per-pixel depth fights (e.g. sword tips vs east/west sheets).

---

## Fake shadows

Shared [`sh_shared_fake_shadow_3d.gdshader`](../project/assets/materials/shaders/shared/sh_shared_fake_shadow_3d.gdshader):

| | Player (`PlayerSpine3d.gd`) | Enemy (`EnemyAgent.gd`) |
|--|---------------------------|-------------------------|
| Silhouette quad | Viewport texture, `use_texture_alpha` | Optional (`fake_shadow_use_silhouette`, default off) |
| Ground blob | Always (separate card) | When not using silhouette |
| Engine shadow maps | Off on cards | Off on cards |

Room/workbench **DirectionalLight** shadows trim only on **High** (or `shading_mode == Lit+Shadows`). Geo `cast_shadow` stays off otherwise.

---

## Trim shaders

| Shader | Purpose |
|--------|---------|
| `sh_surface_trim_worldspace.gdshader` | Floor/wall albedo + ORM (world-space UV) |
| `sh_wall_backface.gdshader` | Translucent interior backface (`next_pass` on wall material) |
| `sh_surface_floor_ao_overlay.gdshader` | Floor vignette AO |
| `sh_fx_fog_of_war.gdshader` | Reveal fog |

**UV policy:** trim materials use `uv1_scale = Vector3.ONE` — never scale trim UVs for room size.

**Texture filter:** trim sheets = **nearest** (pixel crispness). Modular env (door/stair) + static props = **linear with mipmaps** (ORM meshes).

---

## Frame timing (engine policy)

Pathfinder uses **Godot 4.6 defaults** unless `project.godot` overrides them:

| Layer | Rate | Where |
|-------|------|--------|
| **Physics / gameplay** | Fixed **60 Hz** (`physics_ticks_per_second`) | `Player.gd`, `EnemyAgent.gd` — `_physics_process(delta)` with delta-scaled velocity and `move_and_slide()` |
| **Render / presentation** | Variable (monitor refresh, VSync) | `LevelRoot` camera FX, `PlayerSpine3d` decals, UI, `DebugPerf` — `_process(delta)` only |

**Rules:** Do not drive combat or movement from `_process`. Do not add a custom fixed render timestep or global 60 FPS cap unless requirements change (netcode, replay). Spine/combat windows use **track time** or **wall clock**, not frame counts alone.

**VSync:** runtime via [`Settings.vsync_enabled`](../project/autoload/Settings.gd) (default on) — **not** a `project.godot` window/vsync key. No `max_fps` cap; VSync limits present rate.

**Project defaults:** [`project.godot`](../project/project.godot) sets MSAA 3D **2x** as the Medium baseline; Settings quality can override the root viewport at runtime.

**Content FPS** (Spine/sprite export, not engine caps): player ~30 FPS, enemies ~25 FPS — see [godot_topdown_spec.md](godot_topdown_spec.md) and [spine_runetime_notes.md](spine_runetime_notes.md).

---

## Perf / debug (F5)

- **F5** — `DebugPerf` overlay (frame ms, draw calls, spike counters).
- **F4** — CSV to `user://perf/perf_*.csv`.
- **Activity tag `attack`** — set while `Player.is_attacking` (visible in full overlay Mode line).

**Demo attack spike repro:** export demo → F5 ON → F4 ON → idle 5s → walk → light attacks → charge release; compare `frame_ms` / `draw_calls` / `Spikes >16ms` with editor F5. See `.cursor/rules/pathfinder-perf-debug.mdc`.

Likely hotspots (measure before optimizing): Spine SubViewport size/update, `PlayerSpine3d` shadow/decal sync, sword `intersect_shape` during hit window, F2 debug meshes. Do **not** change SubViewport update mode without F5/CSV proof.

---

## Plan review notes (wall split)

- Splitting walls by edge is required so each edge can change `render_priority` and backface independently.
- `depth_test_disabled` on cards is paired with dynamic wall priority — not a standalone “always on top” hack.
- High-risk touch points: `LevelRoot.gd`, `LevelWorkbench.gd`, `RoomPreviewBuild.gd`, `WallDrawOrder3d.gd`.
