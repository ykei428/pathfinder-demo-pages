# Tile size verification (`tile_size_meters`)

Confirms **2D gameplay** (doors, walls, exit tiles) uses the same grid math as **3D preview** (`LevelRoot` / `RoomGridBuild`) when `RoomConfig.tile_size_meters` is not the default **2.0**.

---

## Automated (run after Room / RoomConfig / LevelRoot grid changes)

```powershell
& ".\mcp_local\run-verify-tile-size.bat"
```

Pass line: `VERIFY_TILE_SIZE_OK` / `[verify_tile_size_parity] PASS (2.0, 2.5, 3.0 m)`.

This checks origin and north/south wall lines for 6×6 rooms at 2.0, 2.5, and 3.0 m. It does **not** replace an in-editor walk test after workbench publish.

---

## Manual (after workbench publish with non-2 m tile step)

1. Open **LevelWorkbench** → set **tile size meters** (e.g. **2.5**). If **use native mesh scale** is on, tile step may ignore this — turn it off for the test.
2. **Publish Main** (and Start/Exit if you use custom layouts).
3. Confirm `project/data/room_config.tres` shows `tile_size_meters = 2.5` (or your value).
4. **F5** — walk each wall; try a door transition. Walls should block where the 3D mesh ends; doors should trigger at the opening.

### Pass criteria

- [ ] Published `.tres` has the intended `tile_size_meters`
- [ ] No gap between 3D floor edge and 2D wall colliders
- [ ] Door `Area2D` aligns with the visible door gap (not offset by one tile)

---

## Related

- `project/tests/verify_tile_size_parity.gd` — headless script
- `docs/reviews/full_code_review_phased.md` — Phase 2 verification
- `Room.gd` — reads `room_cfg.tile_size_meters`; `LevelRoot` sets `preview_tile_size_meters` from the same config
