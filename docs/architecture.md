

# Pathfinder – Architecture & Future Plan

> **Last updated:** 2026-03-21  
> Keep this file current — it is the primary onboarding doc for AI agents.

---

## 1. How to Run

- **Gameplay (default):** Open `scenes/MenuRoot.tscn` (main scene) → press **Play** (F5).
- **Direct gameplay (bypass menu):** Open `scenes/LevelRoot.tscn` → press **Play** (F5).
- **Level Workbench:** Open `builds/LevelWorkbench.tscn` in-editor — it is `@tool`, no Play needed.
- **Publish to game:** In the Workbench Inspector → Editor group → click "Publish to Game". This saves a `RoomConfig` resource to `res://data/room_config.tres` which LevelRoot reads at startup.

---

## 2. Project Stack

| Layer | Detail |
|---|---|
| Engine | Godot 4.6.3 stable, GDScript |
| Rendering | Forward+, 1920×1080 design viewport, stretch `canvas_items` + `expand` (see [display_ui_policy.md](display_ui_policy.md)) |
| Characters | 2D Spine Enterprise (runtime only, not @tool-safe) rendered via SubViewport → Sprite3D billboard |
| Environment | 3D modular trim-sheet meshes on a flat XZ plane, tile size 2×2 m. MultiMesh instancing. |
| Physics (gameplay) | **CharacterBody2D** + `move_and_slide` for player/enemies on 2D pixel plane. Gravity disabled. StaticBody2D line-segment wall blockers for 2D collision. |
| Physics (3D env) | StaticBody3D box colliders for 3D environment props. No 3D gameplay physics. |

---

## 3. The 2D → 3D Projection Model

This is the most critical architectural concept. **All gameplay runs in 2D pixel space; the 3D environment is purely visual.**

### Constants (in `autoload/Game.gd`)

```
PIXELS_PER_METER = 100.0          # 1 pixel = 1 cm
get_gameplay_anchor()  # viewport centre at runtime (design fallback 960×540)
```

### Conversion rules

| Direction | Formula |
|---|---|
| **2D → 3D** | `pos_3d.x = (pos_2d.x - ANCHOR.x) / PPM`  ·  `pos_3d.z = (pos_2d.y - ANCHOR.y) / PPM` |
| **3D → 2D** | `pos_2d.x = pos_3d.x * PPM + ANCHOR.x`  ·  `pos_2d.y = pos_3d.z * PPM + ANCHOR.y` |

- All **gameplay speeds and distances** are authored in metres; converted to pixels at `_ready()` via `PPM`.
- Camera FOV is cosmetic only — it **MUST NOT** affect gameplay coordinates.
- **NEVER** use camera raycasting for the 2D→3D projection. Always use the linear PPM mapping above.
- The canonical code is in `Player.gd → _sync_player_3d_projection()`.

### Y-axis (depth) sorting

The 3D card's `sorting_offset` is driven by the 2D Y position via the
centralised helper in `Game.gd`:
```
spine_card.sorting_offset = Game.card_sorting_offset(global_position.y)
# Internally: gameplay_y * Game.CARD_SORT_Y_SCALE  (0.02)
```

---

## 4. Globals (AutoLoads)

| Singleton | Purpose |
|---|---|
| `autoload/Game.gd` | Run-scoped state: gold, RNG, PPM/ANCHOR constants, run lifecycle |
| `autoload/Dungeon.gd` | Floor-scoped state: room graph, clear/discovery flags |
| `autoload/save.gd` | Profile-scoped persistence: 3 slots, JSON in `user://` |
| `autoload/CombatFeedback.gd` | Damage numbers, hit FX |
| `autoload/DebugPerf.gd` | F-key debug overlays |

---

## 5. Scene Map

| Scene | Script | Role |
|---|---|---|
| `builds/LevelWorkbench.tscn` | `LevelWorkbench.gd` (~1850 lines) | **Level Workbench** — @tool editor scene for room shapes, tilesets, props, decals, fog, lighting, camera, publish-to-game |
| `scenes/LevelRoot.tscn` | `LevelRoot.gd` (~1400 lines) | Main game scene — room loading, HUD, camera FX, 3D room preview, 2D wall blockers |
| `scenes/Room.tscn` | `Room.gd` | Runtime combat room (enemies, doors, clear logic, Node2D) |
| `scenes/Player.tscn` | `Player.gd` (~2300 lines) | 2D player controller (CharacterBody2D — HP, stamina, mana, combat) |
| `scenes/PlayerSpine_3d.tscn` | `PlayerSpine3d.gd` (~1050 lines) | 3D Spine character card (SubViewport → Sprite3D, FX decals, health bars) |
| `scenes/EnemyRat.tscn` | `EnemyRat.gd` + `EnemyAgent.gd` | Basic enemy with AI |

---

## 6. Core Loop

1. `Dungeon.generate()` builds a floor graph (6–12 rooms, seeded).
2. `LevelRoot._load_room()` instantiates one room at a time.
3. Combat rooms: enemies spawn → kill all → doors unlock → touch door → transition.
4. Start room always has 4 door options. Backtracking allowed; cleared rooms stay cleared.
5. Exit room advances to next floor.

---

## 7. Level Workbench (`LevelWorkbench.gd`)

The workbench is a self-contained `@tool` scene for visual iteration without pressing Play.  
Scene file: `builds/LevelWorkbench.tscn`. Script: `scripts/LevelWorkbench.gd`.

### Inspector export groups (in order)

| Group | Subgroups | Key exports |
|---|---|---|
| **Tileset** | — | `tileset_name` (auto-discovered from `assets/textures/trims/`) |
| **Layout** | — | `room_shape` (Box/Cross/L/T), `room_width_tiles`, `room_depth_tiles`, `tile_size_meters`, `wall_height_meters`, `arm_width_tiles`, `shape_rotation_deg`, `door_openings_enabled` |
| **Shape Options** | — | Extra shape controls |
| **Camera** | — | `camera_mode` (Auto-fit / Manual), height, tilt, FOV |
| **Material** | — | Normal strength, texture filter |
| **Shading** | — | `shading_mode` (Unshaded / Lit / Lit+Shadows), `uv_mode` (Mesh UV / World-Space UV) |
| **Directional Light** | — | Energy, color, shadow toggle |
| **Spot Light** | — | Energy, color, range, angle, attenuation |
| **Player Light (Torch)** | — | Enabled, energy, color, range |
| **Light Preview Orbit** | — | Orbit speed/radius for preview |
| **Floor AO Overlay** | — | Enabled, strength, radius, color |
| **Props** | — | `props_enabled`, seed, max_count, scale, y_offset, rotation, zone weights, spacing, door clearance |
| **Decals** | — | `decals_enabled`, seed, max_count, opacity, scale range, zone weights |
| **Fog of War** | — | Enabled, color, opacity, reveal radius/softness |
| **Collision** | — | Debug collision box visibility |
| **Scale References** | — | Scale reference helpers |
| **Editor** | Seeds, Random Room | `auto_build_in_editor`, `refresh_now`, `publish_to_game`, `generation_seed`, `props_seed`, `decals_seed`, `random_room`, `min/max_room_tiles` |

### Rebuild pipeline (`_do_rebuild()`)

1. `_resolve_effective_layout()` — resolves random room dimensions if `random_room` enabled; shared by both `_do_rebuild` and `_publish_config`.
2. `_build_tile_grid()` → `_apply_grid_rotation()` — boolean tile grid.
3. `_ensure_meshes()` — loads wall GLB, creates PlaneMesh for floor.
4. Sequential rebuild calls:
   - `_rebuild_room(tile_step)` — floor + wall MultiMesh instances
   - `_rebuild_ao_overlay(tile_step)`
   - `_rebuild_colliders(tile_step)` — 3D StaticBody boxes
   - `_rebuild_props(tile_step)` — prop MultiMesh + individual StaticBody3D colliders
   - `_rebuild_decals(tile_step)` — floor decal Decal3D nodes
   - `_rebuild_fog_of_war(tile_step)`
   - `_rebuild_scale_references(tile_step)`
   - `_rebuild_door_markers(tile_step)`
   - `_rebuild_camera(tile_step)`
   - `_rebuild_lights()`

### Publish flow (`_publish_config()`)

Creates a `RoomConfig` resource populated with **effective** (post-random-resolution) layout, tileset paths, camera, shading, all light params, AO overlay, props, decals, and fog settings. Saves to `res://data/room_config.tres`.

### Key @tool rules

- All deferred work uses `call_deferred()` chains — **never** `await` (crashes the editor).
- Spine runtime is **not** @tool-safe; the workbench uses a static Sprite3D proxy for the player.
- Setting any export var triggers `_do_rebuild()` via property setters for live preview.

---

## 8. Runtime Room Preview (`LevelRoot.gd`)

LevelRoot reads the published `RoomConfig` at startup and builds a 3D preview of the room:

### Startup flow

1. `_ready()` → `_apply_room_config()` — loads `room_config.tres`, populates all cached params, rebuilds tile grid.
2. `_build_static_room_preview()` — sequential calls:
   - `_spawn_preview_floor()` + `_spawn_preview_walls()` — MultiMesh instances
   - `_spawn_preview_door_markers()`
   - `_rebuild_preview_ao_overlay()`
   - `_rebuild_preview_props()` + `_rebuild_preview_decals()`
   - `_rebuild_preview_fog_of_war()`
   - `_rebuild_preview_wall_blockers_2d()` — **2D collision** (see below)

### 2D wall blockers

LevelRoot creates **StaticBody2D** line-segment colliders in pixel space for the 2D physics layer. The pipeline:
1. Iterate the boolean tile grid; identify edges bordering inactive tiles.
2. Skip edges with door gaps (`_is_door_gap_2d()`).
3. Convert 3D metre positions → 2D pixel positions via `GAMEPLAY_ANCHOR` + `PPM`.
4. Merge adjacent horizontal/vertical segments (`_merge_edge_segments()`).
5. Spawn StaticBody2D with CapsuleShape2D per merged segment.

---

## 9. RoomConfig Resource (`scripts/RoomConfig.gd`)

A `Resource` subclass (~165 lines) that bridges Workbench → Gameplay.

| Section | Fields |
|---|---|
| Layout | `room_shape`, `room_width_tiles`, `room_depth_tiles`, `tile_size_meters`, `wall_height_meters`, `arm_width_tiles`, `shape_rotation_deg`, `door_openings_enabled` |
| Tileset | `tileset_name`, 6 texture paths (floor/wall × dif/nrm/orm) |
| Camera | `camera_height`, `camera_tilt_degrees`, `camera_fov` |
| Shading | `shading_mode`, `uv_mode`, `normal_strength` |
| Lights | dir/spot/torch: energy, color, range, angle, attenuation, shadow, indirect_energy |
| AO Overlay | `ao_overlay_enabled`, strength, radius, color |
| Props | `props_enabled`, seed, max_count, scale, y_offset, rotation, zone weights, spacing, door clearance |
| Decals | `decals_enabled`, seed, max_count, opacity, scale range, zone weights |
| Fog | `fog_enabled`, color, opacity, reveal radius/softness, room-revealed state |

**Publish path:** `res://data/room_config.tres`

---

## 10. Material & ORM Pipeline

### Texture conventions

| Asset | Path pattern | Example |
|---|---|---|
| Trim diffuse | `assets/textures/trims/trim_<set>_<surface>_dif.png` | `trim_brick_a_wall_dif.png` |
| Trim normal | `…_nrm.png` | `trim_brick_a_wall_nrm.png` |
| Trim ORM | `…_orm.png` | `trim_brick_a_wall_orm.png` |
| Prop diffuse | `assets/textures/props/prop_<name>_dif.png` | `prop_test_stool_a_dif.png` |
| Prop normal | `…_nrm.png` | `prop_test_stool_a_nrm.png` |
| Prop ORM | `…_orm.png` | `prop_test_stool_a_orm.png` |
| Decal diffuse | `assets/textures/decals/decal_<name>_dif.png` | `decal_common_crack_dif.png` |

### ORM channel packing (CRITICAL)

The ORM texture is a single PNG with 3 channels:

| Channel | Data | Godot property |
|---|---|---|
| **Red** | Ambient Occlusion | `ao_texture_channel = TEXTURE_CHANNEL_RED` |
| **Green** | Roughness | `roughness_texture_channel = TEXTURE_CHANNEL_GREEN` |
| **Blue** | Metallic | `metallic_texture_channel = TEXTURE_CHANNEL_BLUE` |

**CRITICAL:** Godot **multiplies** the texture sample by the base scalar property. You **must** set:
- `mat.metallic = 1.0` — otherwise metallic channel is always zero (Godot default is 0.0).
- Roughness base scalar defaults to 1.0 which is correct.
- If metallic is not set to 1.0, surfaces will show incorrect white specular highlights instead of proper reflections.

### Material creation functions

Both `LevelWorkbench.gd` and `LevelRoot.gd` have these material factories:

| Function | Use case | UV mode |
|---|---|---|
| `_create_standard_material()` | Trim-sheet floor/walls | Mesh UVs, `uv1_scale = Vector3.ONE` |
| `_create_worldspace_material()` | Trim-sheet floor/walls (world-space UV) | Triplanar-lite shader, `texel_density = tile_size_meters` |
| `_create_prop_material()` | FBX/GLB props | Mesh UVs, `uv1_scale = Vector3.ONE` |

**UV policy:** NEVER UV-scale trim-sheet materials. Always `uv1_scale = Vector3.ONE`.

### Shaders

| Shader | Path | Purpose |
|---|---|---|
| World-space UV | `materials/shaders/surface/sh_surface_trim_worldspace.gdshader` | Triplanar-lite trim mapping |
| Floor AO overlay | `materials/shaders/surface/sh_surface_floor_ao_overlay.gdshader` | Vignette-like AO on floor |
| Fog of war | `materials/shaders/fx/sh_fx_fog_of_war.gdshader` | Radial reveal fog |
| Wall backface | `materials/shaders/surface/sh_wall_backface.gdshader` | Prevents backface artifacts on thin walls |
| Player Spine 3D | `materials/shaders/surface/sh_surface_player_spine_3d.gdshader` | Alpha-tested billboard; card Y-sort + wall priority |

**Presentation stack (draw order, shadows, perf):** [3d_presentation_stack.md](3d_presentation_stack.md)

### Environment / Sky

Both `LevelWorkbench.tscn` and `LevelRoot.tscn` include:
- `WorldEnvironment` node with `Environment` resource.
- `ProceduralSkyMaterial` (dark dungeon sky: top=0.1/0.1/0.15, horizon=0.15/0.14/0.16).
- `reflected_light_source = SKY` — IBL reflections from the procedural sky radiance.
- `ambient_light_source = SKY`, energy 0.3.
- `background_mode = Custom Color` (dark), `background_energy_multiplier = 0.0`.

---

## 11. Mesh & Prop Pipeline

### Environment meshes

- **Floor:** Godot `PlaneMesh` (2×2). No GLB — the original floor mesh had unfixable vertex orientation.
- **Wall:** `sm_brick_a_wall.glb` — flat vertical plane (X: −1→1, Y: 0→2, Z ≈ 0). Scaled/rotated per tile edge.
- Both are rendered via **MultiMesh** instancing for performance.

### Props (FBX/GLB)

Props live in `assets/meshes/env/props_static/` and follow the `sm_prop_<name>.fbx` naming convention.

**FBX axis correction (CRITICAL):** FBX meshes are often modelled in Z-up coordinate systems (Blender, 3ds Max). Godot's FBX importer adds a root node rotation (−90° X) to convert to Y-up. When extracting a raw `Mesh` resource from a probed scene instance for use in MultiMesh, this root correction is **not** baked into the mesh vertices. The code **must** walk the node hierarchy from the MeshInstance3D up to the scene root, accumulate the basis transforms, and bake that correction into each MultiMesh instance transform. The relevant code pattern:

```gdscript
var mesh_correction := Basis.IDENTITY
var _walk := probe_mi as Node3D
while _walk != null:
    mesh_correction = _walk.transform.basis * mesh_correction
    if _walk == probe:
        break
    _walk = _walk.get_parent() as Node3D
```

**FBX scale normalisation:** Some FBX meshes are authored in mm or cm scale (e.g. a stool with AABB ~4mm). The code auto-normalises by computing `norm_scale = 1.0 / max_aabb_dimension`, which brings the mesh's longest axis to 1 metre. The user `props_scale` export then acts as a multiplier on top (1.0 = 1m tall prop).

Per-instance transforms are: `mesh_correction.scaled(Vector3.ONE * norm_scale * props_scale)` + yaw rotation.

### Prop placement

Props use a weighted zone system:
1. `_classify_tile_zones()` — categorises tiles into `corner`, `wall`, `scattered` zones based on adjacency.
2. `_select_weighted_positions()` — picks positions from zones using `props_zone_walls`, `props_zone_corners`, `props_zone_scattered` weights with minimum spacing enforcement.
3. Door clearance tiles are excluded (`props_door_clearance_tiles`).

### Adding a new tileset

1. Drop 6 PNGs into `assets/textures/trims/`: `trim_<name>_{floor,wall}_{dif,nrm,orm}.png`
2. Open the Workbench → Tileset group → change `tileset_name` to `<name>`.
3. Auto-discovered by scanning for `trim_*_floor_dif.png`.

### Adding a new prop

1. Export FBX/GLB into `assets/meshes/env/props_static/` named `sm_prop_<name>.fbx`.
2. Drop textures into `assets/textures/props/`: `prop_<name>_{dif,nrm,orm}.png`.
3. The workbench `_find_first_prop_mesh()` discovers the first `sm_prop_*` file alphabetically.
4. Material is auto-built from texture convention: `sm_prop_<name>` → strip `sm_` → look for `prop_<name>_dif.png` etc.

---

## 12. MultiMesh Instancing

All repeated geometry (floors, walls, props) uses `MultiMesh.TRANSFORM_3D` for draw-call batching:

```
_spawn_multimesh(parent, node_name, mesh, material_override, transforms_array)
```

- Creates a `MultiMeshInstance3D` with one draw call for all instances.
- Material override applied to the entire batch.
- Colliders are still individual `StaticBody3D` nodes (not batched).

---

## 13. State Ownership Boundaries

| Scope | Owner | Lifetime | Data |
|---|---|---|---|
| Floor | `Dungeon` | One floor gen → next floor gen | Room graph, per-room clear/discovery, spawn seeds |
| Run | `Game` | Run start → run end | Gold, RNG, loot pickups (run inventory planned) |
| Profile | `Save` | Persistent across runs | Banked currency, meta XP, character level, stat allocations; 3 slots |
| Hub | `Game` (`SessionMode.HUB`) | Menu → hub; run end → hub | Character room; no active `Dungeon` graph |

---

## 14. Persistence

- Backend: JSON files in `user://` via `autoload/save.gd`.
- 3 profile slots with separate progression.
- Manual saves only between runs (character-room pause menu, Esc); returning to the title menu autosaves. No mid-run *manual* save and no run resume.
- Run-end (death) commits rewards and autosaves; player returns to character room automatically.
- Hub enter triggers an additional autosave (idempotent after run-end autosave).
- Crash-safe banking: each time a floor is completed, the run's earned gold/XP and reached floor are written to a per-slot `pending_run` ledger (`record_run_checkpoint`). The run itself is never resumable. The ledger is reconciled exactly once — cleared by `apply_run_rewards` on a clean run-end, or applied at next launch by `recover_pending_run` (full ratio, neutral exit) if the previous session crashed mid-run. In-floor progress earned after the last checkpoint is not recovered (conservative). No save-scum surface (a kill/crash still ends the run).
  - **Zero-floor-clear crash window:** `apply_run_rewards` updates the in-memory profile; the durable disk write happens in the following `autosave_end_of_run`. If a run ends (incl. death) before clearing any floor — so no `pending_run` checkpoint exists — and the process is killed in the gap between those two calls, that run's rewards are lost. This window is tiny and conservative by design (worst case: lose one short, floor-1 run); floor checkpoints close it for any run that clears at least one floor.
- Profile v7 fields: `character_level`, `stat_points_unspent`, `stat_allocations` (7 primary stats — see [character_stats.md](character_stats.md)), `inventory_owned`, `loadout` (2 slots; hub equip only), `best_floor_reached`, `total_play_seconds`, `pending_run` (in-progress run ledger; `{}` when none). See `CharacterProgression`, `CharacterStatFormulas`, `EquipmentCatalog`.
- Hub Esc → CHARACTER (stat spend + loadout) → STORE (buy gear, reset stats for gold).

---

## 15. Future Plan / Roadmap

### Near-term (next milestones)
1. **Tileset variety** — Create additional trim-sheet sets (stone, wood, dungeon_dark) and validate the auto-discovery pipeline.
2. **More props** — Add barrel, torch, chest FBX meshes. Currently only `sm_prop_test_stool_a.fbx` exists.
3. **Door mesh integration** — Replace placeholder wall gaps with dedicated door-frame and door-panel meshes.
4. **Lighting presets** — Named lighting presets in `RoomConfig` that the workbench can preview and publish.
5. **Multi-prop support** — Currently the workbench places only the first discovered prop mesh. Extend to support multiple prop types per room with per-type weights.

### Mid-term
6. **Spine player in gameplay** — Finalize the PlayerSpine_3d card pipeline: SubViewport → Sprite3D billboard with alpha-tested shader, depth-correct sorting, health/stamina/mana bars.
7. **Enemy variety** — Add 2–3 more enemy types beyond the rat.
8. **Difficulty scaling data** — Move scaling knobs to a single editable data source under `project/data/`.
9. **Minimap HUD** — Discovered rooms, cleared state, exit marker.

### Long-term
10. **Run inventory / items / buffs** — Loot pickups + `LootTable` data exist; run bag/hotkeys still planned in `Game`.
11. **Meta progression** — Character room hub, level/XP, stat points (inventory/loadout deferred).
12. **Boss rooms** — Special room type with unique layout, boss enemy, and clear reward.
13. **Audio pass** — Per-room ambient variations, combat music layers, spatial SFX.
14. **Polish & export** — Performance profiling, Steam build pipeline.

---

## 16. Conventions for AI Agents

### DO

- **Naming:** Strict `lowercase_snake_case` with prefixes (`sm_`, `trim_`, `mat_`, `sh_`, etc.). See `.github/instructions/pathfinder-chat.instructions.md` for full table.
- **UV policy:** Always `uv1_scale = Vector3.ONE` unless explicitly told otherwise.
- **ORM materials:** Always set `metallic = 1.0` when assigning a metallic texture, so the blue channel actually controls metallicity.
- **FBX props:** Always capture the FBX root→mesh node basis correction and bake it into MultiMesh transforms.
- **Speeds/distances:** Define in metres. Derive pixels at startup via `Game.PIXELS_PER_METER`.
- **2D→3D projection:** Use the linear PPM mapping. Never camera raycasting.
- **@tool safety:** Never use `await` in @tool scripts. Always `call_deferred`.
- **Commit flow:** Bump `project.godot` `config/version` for player-facing changes; commit subject = `v <semver> - <summary>`. Never commit/push unless user says `push now` (use `/push-now`).
- **Revert safety:** Never run revert/restore/reset operations unless explicitly requested. Always backup first.

### DON'T

- **No hacks or workarounds.** If a proper fix is possible, do it. If not, explain why and propose alternatives.
- **Never switch player rendering to 2D sprite fallback** as a draw-order workaround. Must remain 3D-card-based.
- **Never UV-scale trim-sheet materials** for room previews.
- **Never hardcode pixel values** for gameplay distances. Always derive from metres × PPM.
- **Door width:** `2×2` = tile dimensions only; default door opening = one center tile per side unless explicitly widened.