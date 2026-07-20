# Pathfinder — game overview

Top-down **2.5D dungeon roguelite** prototype (Godot 4.6.3). Visual inspiration: *Binding of Isaac*, *Morsels*. Gameplay is **2D**; environments and Spine characters are **3D** presentation.

## Core loop

1. Title menu → pick profile slot (New / Continue / Load)
2. **Character room** hub — spend stat points, equip gear, open store
3. Enter dungeon (south door) → procedural floors, combat, loot
4. Death or exit → bank gold/XP to profile → return to hub
5. Meta XP → levels → **stat points** → repeat

Manual save: hub pause (Esc) or autosave on hub enter / run end. No mid-run *manual* save and no run resume — but earned gold/XP and the reached floor are banked per floor completed, so a crash mid-run still credits progress up to the last completed floor.

## Character stats (v1)

Seven **primary stats** upgraded with level-up points: Vitality, Intelligence, Strength, Agility, Dexterity, Defense, Luck. They feed **derived** combat and reward values (HP, HP regen, damage, armor, shield, crit chance/damage, move speed, gold gain, etc.).

Details: [character_stats.md](character_stats.md) · full design: [ai/deep-research-stats.md](ai/deep-research-stats.md)

## Further reading

| Doc | Content |
|-----|---------|
| [architecture.md](architecture.md) | Code structure, save ownership, 2D↔3D |
| [godot_topdown_spec.md](godot_topdown_spec.md) | Art / world constraints |
| [loot_tuning.md](loot_tuning.md) | Drop tables, luck hooks |
