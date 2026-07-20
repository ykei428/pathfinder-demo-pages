# Display and UI policy (PC landscape)

**Design resolution:** 1920×1080 (16:9). Author HUD spacing and fonts at this size.

**Stretch (project):**

| Setting | Value |
|---------|--------|
| `window/stretch/mode` | `canvas_items` |
| `window/stretch/aspect` | `expand` |
| `window/size/resizable` | `true` |

The window fills the monitor. Internal layout coordinates stay 1920×1080; Godot scales to the OS window.

**Upscale on large monitors:** UI and canvas **scale to the OS window pixel size** (internal layout stays 1920×1080). A 3840×2160 window ≈ 2× UI size.

**F5 in the editor (embedded — most common):** If **Embed Game on Next Play** is on and **Embedded Window Sizing → Fixed Size** is selected (Godot play menu), the game panel is **always 1920×1080** inside the editor. Stretch only fills that panel — you will see `1920×1080` in the corner on a 4K monitor. This is **editor behaviour**, not the shipped game.

To preview 4K / large-window scaling in-editor:

1. Play menu (▶) → turn off **Embed Game on Next Play** (separate window), **or**
2. Keep embed but set **Embedded Window Sizing → Stretch to Fit** (uses dock size; still not full 4K unless the dock is huge).

**Exported / standalone `.exe`:** Opens its own OS window. `DisplayLayout` grows the default 1920×1080 launch size to your monitor (up to **3840×2160** on 4K, 2× cap). That is what players get — not the embedded fixed panel.

**Check:** Separate play window or demo `.exe` → F5 stats / `DisplayServer.window_get_size()` should be ~3840×2160 on 4K, not 1920×1080.

**Cap:** `DisplayLayout` autoload limits automatic upscale to **2×** over design (`MAX_CONTENT_SCALE = 2.0`, i.e. up to ~3840×2160 effective). On 8K and above, UI stays at 2× size (extra pixels become margin), not 4×.

To change: edit `project/autoload/DisplayLayout.gd` or remove the autoload to allow unlimited fractional upscale.

**Safe inset:** HUD stats use `HUD_SAFE_INSET` (48px) from the top-left so text is not clipped by the editor game frame or window chrome.

## Gameplay vs UI

| Layer | Space | Rule |
|-------|--------|------|
| Gameplay / 2D physics | Pixel plane anchored at viewport centre | `Game.get_gameplay_anchor()` — **not** fixed 960×540 at runtime |
| PPM | `Game.PIXELS_PER_METER` (100) | Never scale for “responsiveness” |
| Camera FOV | Cosmetic | Never used for gameplay projection |
| HUD | `CanvasLayer` + `Control` anchors | Corner pins + `HUD_MARGIN` (24px at design scale) |

Do not use camera raycasts for 2D↔3D. World-attached bars use `Game.world_to_canvas()`.

## Supported aspect ratios

| Tier | Target | QA |
|------|--------|-----|
| Primary | 16:9 @ 1920×1080 | Default |
| Minimum | 1280×720 (16:9) | Required before HUD ship |
| Laptop | 1920×1200 / 1280×800 (16:10) | Recommended |
| Ultrawide | 2560×1080 / 3440×1440 (21:9) | Recommended — corners pinned, extra width is gameplay |
| 4:3 | — | No bespoke layout; expand gives more vertical room |

## HUD layout (LevelRoot)

- **Top-left:** `MarginContainer` → `VBoxContainer` (HP, SP, MP, gold, floor, XP, charge).
- **Top-right:** version string (debug builds).
- **Bottom-right:** minimap (`Minimap` control, local `_draw`).
- **Centre-top:** popup messages (max width capped in `HUD.gd`).

## Gamepad hints and labels

- **Gameplay:** `project.godot` InputMap uses SDL abstract `JoyButton` indices (`device = -1`). Xbox A, PlayStation Cross (×), and Switch south face all map to the same index — no per-brand preset rows.
- **UI labels:** [`project/scripts/GamepadLabels.gd`](../project/scripts/GamepadLabels.gd) picks Xbox / PlayStation / Nintendo names from `Input.get_joy_name()` for the Controls screen and start-room floor hints.
- **Control prompts:** Settings → **Control prompts** → **Auto** (default): keyboard labels when no pad is connected; gamepad labels when a pad is connected. Manual Keyboard / Gamepad overrides stay available. The Controls screen shows the detected pad type (Xbox, PlayStation, etc.) in small text under the title.

Default combat pad layout (same indices on all standard pads): **A / × attack**, **B / ○ dodge forward**, **Y / △ dodge back**, **LB / L1 block**, left stick move.

## Menu and gamepad input

Menus and hub overlays share one UI for mouse, keyboard, and gamepad. **Focus** is the shared cursor for keyboard and pad; mouse click should stay in sync (`grab_focus()` on activate).

| Input | Affordance |
|-------|------------|
| Mouse | Click activates; hover may preview (optional). |
| Keyboard | Tab / arrows move focus; Enter activates. |
| Gamepad | D-pad or stick (`ui_*`) moves focus; **A** (`menu_accept`) activates; **B** (`menu_back`) goes back. |

Do **not** bind `menu_accept` to Space if another gameplay action uses Space on the same binding policy. Title menus: [`MenuRoot.gd`](../project/scripts/ui/menu/MenuRoot.gd). Hub overlays: [`HubInputHelper.gd`](../project/scripts/ui/HubInputHelper.gd).

### Tooltips vs detail panes

Godot `tooltip_text` is **hover-only** — it does not show on gamepad focus.

| Tier | Mouse | Gamepad / keyboard |
|------|-------|-------------------|
| Primary | Visible on row/card (name, price, stat) | Same |
| Secondary | Optional `tooltip_text` on hover | Detail pane or selection panel updates on focus |
| Tertiary | Hover-only OK | Skip |

Do not use mouse-position `PopupMenu` as the only path for an action (equip, slot pick). Use a centered, focusable modal ([`UiEquipPicker.gd`](../project/scripts/ui/kit/UiEquipPicker.gd)).

### New screen checklist

1. Interactive controls: `focus_mode = FOCUS_ALL`; decorative labels: `FOCUS_NONE`.
2. On open: `call_deferred` grab focus on first meaningful control ([`HubInputHelper.grab_first_focus`](../project/scripts/ui/HubInputHelper.gd)).
3. Custom panels (cards, rows): handle `ui_accept` in `_gui_input`; wire `focus_entered` → selection when a detail pane or action button depends on it.
4. Back: `menu_back` / `ui_cancel` closes overlay or pops stack (hub pickers close before their parent).
5. Binding labels: Settings → **Control prompts** + [`GamepadLabels.gd`](../project/scripts/GamepadLabels.gd) on the Controls screen and in-world hints — never `Input.get_connected_joypads().is_empty()` for policy.
6. QA: complete the screen flow with gamepad only (no mouse).

## Manual QA checklist

1. F5 @ 1920×1080 — player centred, room load, HUD corners.
2. Resize window — HUD stays in corners; no drift vs 3D card.
3. 1280×720 — labels readable, minimap not overlapping stats.
4. Ultrawide — HUD on screen edges; world shows wider view (camera fit).

Headless smoke: `.\mcp_local\run-godot-headless.ps1 -ProjectPath ".\project"`
