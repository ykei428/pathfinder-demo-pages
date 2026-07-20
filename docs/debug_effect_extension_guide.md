# Debug + Effect Extension Guide

This guide explains the current safe pattern for extending debug actions and decal/effect behavior without introducing regressions.

## Goals
- Preserve gameplay/debug parity.
- Keep debug menu behavior stable.
- Avoid duplicate signal callbacks and stale visual state.

## Debug Menu Safety
- Keep existing shortcut behavior unchanged:
  - `F2` hitbox toggle
  - `F4` CSV toggle
  - `F5` stats mode toggle
  - `F6` debug menu
- Add new debug buttons in `project/scripts/HUD.gd` via `_ensure_debug_menu()` and route actions through `_run_debug_player_action(...)`.
- For any new debug method, implement it on `project/scripts/Player.gd` and guard cross-script calls with `has_method(...)`.
- HUD player signal wiring should use named methods + explicit disconnect/reconnect (no inline lambdas in reconnect paths).

## Effect/Decal Safety
- Live effect ownership:
  - Gameplay state and timing: `project/scripts/Player.gd`
  - 3D decal/shockwave presentation: `project/scripts/PlayerSpine3d.gd`
- Keep state transitions explicit on deactivation:
  - clear transient flashes
  - reset runtime multipliers
  - clear one-shot guards
- Avoid adding hard-disabled branches (`if false`) for experiments. Prefer a real feature flag if needed.

## Adding a New Debug FX Test
1. Add button in `HUD.gd` under an existing section.
2. Route click to `_run_debug_player_action("<new_method>", "Debug: <label>")`.
3. Add `<new_method>` in `Player.gd` and call into `player_3d` with `has_method(...)` checks.
4. Verify behavior with menu open and closed.

## Adding a New Decal Effect
1. Add exported tuning vars in `PlayerSpine3d.gd` near related effect group.
2. Keep setup/sync/clear functions separate.
3. Ensure alpha/emission behavior is consistent with current dual-tint shader behavior.
4. Ensure cleanup removes all spawned nodes to avoid leaks.

## Quick Regression Checklist
- No `get_errors` issues in edited scripts/shaders.
- Debug menu opens/closes and pause behavior still matches checkbox state.
- Existing FX tests still work.
- Bloodlust start/end and charge release visuals still clear correctly.
