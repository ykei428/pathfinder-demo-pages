# Room Generation Notes

> **Last updated:** 2026-03-19

---

## Two Room-Preview Systems

### 1. Level Workbench — `TestRoom3D.tscn` / `TestRoom3D.gd` (@tool, editor-only)

The primary art-direction and iteration tool. Generates rooms in the 3D editor without pressing Play.

**Current capabilities:**
- Boolean tile grid: `Array[bool]`, index = `z * width + x`.
- Shapes: Rectangle, L-Shape, T-Shape, Cross.
- Size presets: 6×6, 8×6, 10×10, 12×8, 14×14, Custom.
- Shape rotation: 0°, 90°, 180°, 270° (grid rotation, not node rotation).
- Seed-based random room generation.
- Tileset auto-discovery: scans `assets/textures/trims/` for `trim_<name>_floor_dif.png`.
- Camera: Auto-fit (from grid bounds) or Manual (height, tilt, FOV).
- Lighting: DirectionalLight3D + SpotLight3D + optional OmniLight3D torch.
- Shading: Unshaded / Lit / Lit+Shadows.
- UV: Mesh UV / World-Space UV (triplanar-lite shader).
- Player proxy: Sprite3D billboard of `player_fighter.png` (Spine not @tool-safe).
- Publish button: saves `RoomConfig` resource to `res://data/room_config.tres`.

**Floor mesh:** Always Godot `PlaneMesh` (2×2). The floor GLB was deleted (vertex orientation was broken and unfixable without re-export).

**Wall mesh:** `sm_brick_a_wall.glb` — flat vertical plane (X: −1→1, Y: 0→2, Z ≈ 0). `_compute_tile_scale` guards against near-zero Z axis (skip divide if size < 0.01).

**UV fix:** `texel_density` in the world-space shader must receive `tile_size_meters` (e.g. 2.0), not 1.0. This ensures 1 tile = exactly 1 full trim sheet with no UV repeats.

**@tool safety:** All deferred work uses `call_deferred()` chains — never `await`.

### 2. Gameplay Preview — `LevelRoot.tscn` / `LevelRoot.gd` (@tool + runtime)

A secondary preview system inside the main game scene. Provides a 3D room preview under `World3D/3D Models/RoomPreview` plus 2D `StaticBody2D` wall blockers for runtime collision.

**Key exports (Inspector → Room Preview):**
- `preview_enabled`, `preview_room_width_tiles`, `preview_room_depth_tiles`, `preview_tile_size_meters`
- `preview_size_multiplier` (visual scaling factor, default ~2.667)
- `preview_use_native_mesh_scale`, `preview_mesh_scale_multiplier`
- `preview_only_play_mode` — skips dungeon bootstrap, shows only the preview
- `load_from_config` — one-shot button that reads `RoomConfig` from `res://data/room_config.tres`, applies layout + tileset overrides, and rebuilds preview

**Current limitation:** LevelRoot's preview is locked to rectangular rooms only. Non-rectangular shapes (L, T, Cross) exist only in TestRoom3D for now.

---

## RoomConfig Resource (`scripts/RoomConfig.gd`)

Bridges Workbench → Gameplay. Stores layout, tileset paths, camera, shading, and lighting settings. Published via TestRoom3D's "Publish to Game" button; consumed via LevelRoot's "Load from Config" button.

Save path: `res://data/room_config.tres`

---

## Tileset Pipeline

| Component | Path Pattern |
|---|---|
| Diffuse | `assets/textures/trims/trim_<set>_<surface>_dif.png` |
| Normal | `assets/textures/trims/trim_<set>_<surface>_nrm.png` |
| ORM (packed) | `assets/textures/trims/trim_<set>_<surface>_orm.png` |

Only `brick_a` exists currently. To add a new tileset, drop 6 PNGs and the workbench auto-discovers it.

---

## Door Openings

- Default: one center-tile gap per side (4 openings total).
- `2×2` = tile dimensions, NOT a 2-wide doorway.
- Dedicated door-frame/panel meshes are planned but not yet created.

## Wall Blockers (2D, runtime only)

- `PreviewWallBlockers2D` node under LevelRoot creates `StaticBody2D` segments at runtime.
- Center openings are preserved to match 3D door gaps.
- Blockers are NOT spawned in editor (`Engine.is_editor_hint()` guard).
- Tune thickness: `preview_2d_blocker_half_thickness_px`.

---

## Future: Runtime Procedural Rooms

The boolean-grid room builder in `TestRoom3D` is intended to be ported into the gameplay `Room` scene so rooms are procedurally shaped at runtime (not just rectangles). The `RoomConfig` resource (or a slimmed-down variant) will feed shape/seed data from `Dungeon` per-room. This is the next major milestone after tileset variety and prop placement.
