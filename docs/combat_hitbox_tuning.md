# Combat hitbox tuning (guardrails)

**Status:** Tuned May 2026 — F2 3D overlay + 2D gameplay shapes aligned.  
**Do not change casually.** Read this before editing combat hitboxes or attack approach AI.

## Two layers (do not mix)

| Layer | What it is | Changes gameplay? |
|-------|------------|-------------------|
| **Gameplay** | `Player.tscn` / `EnemyRat.tscn` → `Hurtbox`, `SwordHitbox`, body `CollisionShape2D` | **Yes** — damage, blocking, physics |
| **F2 debug** | `CombatHitboxDebug3d.gd` on `PlayerSpine3d` + each enemy card | **No** — visual only (unless you also move 2D shapes) |

**Rule:** Tuning in Remote inspector (`hurtbox_*`, `sword_hitbox_offset_*`) affects gameplay when read by `Player.gd` / `EnemyAgent.gd`. F2 overlay must follow the same sources — do not add a second offset path in `CombatHitboxDebug3d.gd`.

## Coordinate mapping (2.5D)

- Gameplay plane: **CharacterBody2D** X/Y (pixels at runtime via `Game.PIXELS_PER_METER` = 100).
- World 3D: **X** ← gameplay X, **Z** ← gameplay Y, **Y** ← height / floor / card torso.
- **Never** use camera raycasts for gameplay ↔ debug alignment.
- Player card origin = **feet** (`floor_height`). Enemy card origin = **centre** (half lift from `card_floor_height`).

## Key scripts (owners)

| Concern | Owner |
|---------|--------|
| Player hurtbox 2D position | `Player._sync_hurtbox_facing()` |
| Player sword 2D window | `Player._update_sword_hitbox_position()`, `_run_sword_hitbox_window()` |
| F2 red / orange / yellow meshes | `CombatHitboxDebug3d.gd` |
| F2 toggle | `HUD.gd` + `Game.combat_hitbox_debug_enabled` |
| Rat attack spacing / melee AI | `EnemyAgent.gd` (not hitbox shapes) |

## Current tuned exports (code defaults)

### Player hurtbox (red F2 + 2D)

| Export | Value | Notes |
|--------|-------|--------|
| `hurtbox_torso_width_fraction` | 0.13 | |
| `hurtbox_torso_height_fraction` | 0.55 | |
| `hurtbox_torso_center_height_fraction` | 0.38 | Torso Y from feet: `card_h × 0.5 × fraction` |
| `hurtbox_lateral_offset_m` | 0.0 | Gameplay X only |
| `hurtbox_depth_offset_m` | −0.15 | F2: `−= basis.z × value` (toward camera) |

### Player sword (yellow F2 + 2D, active window only)

| Export | Value | Notes |
|--------|-------|--------|
| `sword_hitbox_offset_right_m` | (0.30, 0) | Gameplay X → world X |
| `sword_hitbox_offset_left_m` | (−0.30, 0) | |
| `sword_hitbox_offset_front_m` | (0, 0) | gameplay Y → world Z |
| `sword_hitbox_offset_back_m` | (0, 0) | |
| `sword_hitbox_extend_downward_percent` | 0.15 | Must use facing-relative extend in `_update_sword_hitbox_position` — **do not** add raw `shape_pos.y` only |

Reach along swing: rectangle half-width (~87.5 px) via `_sword_hitbox_shape_local_origin()` — not the direction exports.

### Rat hurtbox (orange F2 + 2D)

| Export | Value |
|--------|-------|
| `hurtbox_torso_width_fraction` | 0.286 |
| `hurtbox_torso_height_fraction` | 0.28 |
| `hurtbox_torso_center_height_fraction` | 0.28 |
| `hurtbox_depth_offset_m` | −0.30 |
| `card_screen_offset` (EnemyRat.tscn) | (−5, −75) px — affects body band math |

## Do / don't

**Do**

- Tune via **Remote inspector** while **F5 + F2** on a combat room.
- Use **`/plan-review`** before structural refactors in `Player.gd`, `CombatHitboxDebug3d.gd`, or `EnemyAgent` melee AI.
- Fix **one authority** per axis (gameplay shape → F2 reads it; one combat distance helper for AI).
- Run headless after script changes: `.\mcp_local\run-godot-headless.ps1 -ProjectPath ".\project"`

**Don't**

- Re-add HUD `_draw()` 2D hitbox overlays (removed; F2 3D only).
- Duplicate lateral offset in F2 **and** `HurtboxShape` for player (F2 follows shape; no extra `basis.x` lateral).
- Apply sword extend offset on raw **shape local Y** without rotating with facing (breaks up/down attacks → false X shift).
- Change `CollisionShape2D` sizes/positions in `.tscn` unless intentionally retuning **gameplay** hit detection.
- Patch alignment with per-animation `if attack/start` branches — use exports + shared helpers.

## Known pitfalls (regression checklist)

1. **Player feet vs card centre** — hurtbox Y must use feet + upper half of card, not full texture height.
2. **Enemy depth** — `hurtbox_depth_offset_m` sign: negative export → toward camera via `−= basis.z × m`.
3. **Rat AI** — use `_get_combat_distance_to_player()` (min of body + origin); don't chase with origin-only distance.
4. **Sword up/down** — front/back offsets are (0,0); sideways reach is shape placement + rotation, not Z export.

## Related

- `.cursor/rules/pathfinder-combat-hitbox.mdc` — agent rule when editing these files
- `docs/architecture.md` §16 — fix quality for agents
