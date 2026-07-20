# Loot tuning

Data-driven pickups: items, drop chances, and gold value scaling.

**Related:** [architecture.md](architecture.md) · [floor_scaling.tres](../project/data/floor_scaling.tres) (gold **amount** per coin)

## Where to edit

| What | File |
|------|------|
| Item identity + effects + placeholder color | `project/data/items/*.tres` |
| Drop rates (per kill) | `project/data/loot/default_loot_table.tres` |
| Gold value per coin | `project/data/floor_scaling.tres` → `get_gold_per_kill(floor)` |

Do not duplicate percentages in this doc — tune in the `.tres` files only.

## Drop model

Each row in a loot table is a **`LootDropEntry`**. On enemy death, `LootTable.roll_drops()` runs **one independent random roll per enabled row** (not a single weighted hat). That allows:

- Gold every kill (`drop_chance = 1.0`)
- Potions sometimes (`drop_chance ≈ 0.10`)
- Super-rares later (`drop_chance` very low, `enabled = false` until the item exists)

Modifiers passed into rolls:

- `global_multiplier` — from `LootTable.global_drop_multiplier` (and later floor/luck systems)
- `luck_multiplier` — from profile **Luck** primary stat at run start (`Game.run_luck_rare_multiplier` × `run_luck_drop_bonus`). Applied in `LootDropEntry.get_effective_drop_chance()` for **uncommon+** rarities only (common/gold rows unchanged).

Per-entry floor scaling: `drop_chance_per_floor`, capped by `drop_chance_cap`.

## Adding a new item

1. Duplicate an item under `project/data/items/` (e.g. `health_potion_large.tres`).
2. Set `id`, `effects`, placeholder visuals.
3. Add a `LootDropEntry` sub-resource row in `default_loot_table.tres` (or a new table for bosses/chests).
4. No changes to `Room.gd` unless you use a different table path.

## Gold flow

- Kill → roll table → spawn `gold_coin` pickup → player walks over it → `Game.add_cash` → run end banks to profile `Save.cash`.
- Coin pickup reads `gold_amount` from roll context (from floor scaling), not a fixed value on `ItemDef`.

## Equipment security (dungeon)

- **Common** equipment: added to profile inventory on pickup (unchanged).
- **Uncommon / rare / legendary** equipment: staged in `Game.run_unsecured_equipment` until **floor clear** (`record_run_checkpoint` commits to profile).
- **Death:** staged items are discarded (not granted); run gold/XP still bank at 100%.
- Pickup float text for staged items: `Unsecured: <name>`.

## Code map

| Role | Script |
|------|--------|
| Roll API | `project/scripts/loot/LootTable.gd` |
| Apply effects | `project/scripts/loot/LootEffects.gd` |
| World pickup | `project/scripts/loot/LootPickup.gd` |
| Spawn on kill | `project/scripts/Room.gd` |

Cross-script types use `preload("res://…")` consts (e.g. `ItemDefScript`) so Cursor LSP and Godot agree — see `.cursor/rules/pathfinder-gdscript-verify.mdc` → **New script modules**.
