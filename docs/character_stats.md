# Character stats (v1)

Quick reference for the primary → derived stat system. **Canonical design:** [ai/deep-research-stats.md](ai/deep-research-stats.md).

## Primary stats (level-up points)

| Stat | Purpose | Main derived effects |
|------|---------|----------------------|
| **Vitality** | Survivability | Max HP, HP regen/s, healing received |
| **Intelligence** | Mana / abilities | Max mana, mana regen |
| **Strength** | Reliable damage | Damage multiplier → sword damage (optional life-steal hook, off by default) |
| **Agility** | Movement & dodge | Move speed (m/s), dodge cooldown |
| **Dexterity** | Attack tempo & crit | Attack speed, crit chance, crit damage |
| **Defense** | Mitigation | Armor → damage reduction (diminishing returns), shield pool |
| **Luck** | Rewards | Rare drop weight, drop bonus (non-common rows only), gold gain |

Stamina is **not** a primary stat in v1; dodge still costs stamina from base player tuning.

**V2 additions** (extend the existing 7 stats — no new primary stats): Vitality HP regen + healing received, Dexterity crit-damage progression, Defense shield pool, Luck gold gain. Life steal and XP gain are gear-driven (equipment effects `bonus_lifesteal`, `bonus_shield`, `run_xp_multiplier`). Deferred: Intelligence mana-cost/ability efficiency and Status Power / Control (need ability and status-effect systems first). See [ai/deep-research-stats.md](ai/deep-research-stats.md).

## Derived formulas (summary)

Tuning lives in `project/data/character_primary_stats.tres`. Code: `CharacterStatFormulas.gd`.

- **Armor DR:** `damage_taken = incoming × 100 / (100 + armor)`, minimum 1 after armor
- **Shield:** Defense-derived pool absorbs damage **after** armor, **before** HP; refilled on build / hub revive / floor clear
- **Crit damage:** base **150%**, +`dexterity_crit_damage_per_point` per Dexterity point, capped by `max_crit_damage_multiplier`
- **HP regen:** Vitality-derived HP/s, gated to start `hp_regen_delay_after_damage` seconds after the last hit
- **Soft caps:** points above cap count at 50% effectiveness (see design doc)

## Code map

| Role | Path |
|------|------|
| Tunable numbers | `project/data/character_primary_stats.tres` |
| Formulas | `project/scripts/CharacterStatFormulas.gd` |
| XP / allocate / reset | `project/scripts/CharacterProgression.gd` |
| Apply to player | `project/scripts/Player.gd` → `apply_character_build()` |
| Hub UI | `project/scripts/ui/HubCharacterSheet.gd` |
| Save | `stat_allocations` keys in profile v5; legacy `max_hp`/`damage` migrate to vitality/strength |

## Active vs passive defense

- **Armor (Defense stat):** passive reduction on hits that connect
- **Block / dodge:** active mechanics; successful block still negates damage (stamina-gated)
