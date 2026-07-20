\# Godot Runtime Notes



Purpose: Describe how this project wants to use Godot, so code and scenes stay consistent.



\## Version and basics



\- Engine: \*\*Godot 4.6.3\*\*

\- Language: \*\*GDScript\*\*

\- Project type: 3D project used as a \*\*2.5D topdown\*\* dungeon game

\- Target platform: Steam / PC  

&nbsp; - Secondary: mobile support later, low priority



\## World and coordinates



\- Gameplay happens on a \*\*flat XZ plane\*\*

\- No vertical traversal (no stairs, plateaus or jump gameplay)

\- Tile size in world space: \*\*2×2 meters\*\*

\- Rooms are aligned on this 2×2 meter tile grid



\## Scene structure conventions



These are starting points that can be refined as the project grows.



\### Player scene



\- Scene: `Player.tscn`

\- Root node:

&nbsp; - `CharacterBody3D` named `Player`

\- Children (suggested):

&nbsp; - `CollisionShape3D`  

&nbsp; - `Sprite3D` or `AnimatedSprite3D` for the Spine driven or sprite sheet driven visual

&nbsp; - Optional `Area3D` + `CollisionShape3D` for melee hitboxes or interaction

\- Script:

&nbsp; - `Player.gd` attached to the root `CharacterBody3D`



\### Enemy scenes



\- One scene per enemy type, for example:

&nbsp; - `Enemy\_Slug.tscn`, `Enemy\_Fly.tscn`, etc

\- Root:

&nbsp; - `CharacterBody3D` or custom enemy base node

\- Children:

&nbsp; - `CollisionShape3D`

&nbsp; - `Sprite3D` / `AnimatedSprite3D` for visuals

&nbsp; - `Area3D` for hit or aggro zones if needed



\### Room / level scenes



\- Room scene example: `Room\_Base.tscn`

\- Root:

&nbsp; - `Node3D` or dedicated `Room3D` script

\- Children:

&nbsp; - One node to hold geometry tiles (floors, walls, doors)  

&nbsp;   - e.g. `Tiles` as a `Node3D`

&nbsp; - One node to hold gameplay entities (enemies, props, pickups)  

&nbsp;   - e.g. `Entities` as a `Node3D`

&nbsp; - One optional node for triggers, doors and exits  

&nbsp;   - e.g. `Triggers` as a `Node3D`



\## Movement and physics



\- Player and enemies:

&nbsp; - Use `CharacterBody3D`

&nbsp; - `gravity` disabled

&nbsp; - Movement restricted to XZ plane

\- Movement:

&nbsp; - Use `move\_and\_slide()` (or Godot 4 equivalent pattern) with:

&nbsp;   - Input mapped to `move\_left`, `move\_right`, `move\_up`, `move\_down`

&nbsp;   - Velocity updated each physics frame based on input

\- Environment:

&nbsp; - Use static bodies / `StaticBody3D` with `CollisionShape3D`

&nbsp; - Colliders aligned to the 2×2 meter tile grid for floors, walls and doors



\## Input conventions



\- Use \*\*InputMap\*\* actions, not raw keycodes

\- Suggested actions:

&nbsp; - `move\_up`, `move\_down`, `move\_left`, `move\_right`

&nbsp; - `attack`

&nbsp; - `dodge`

&nbsp; - `interact`

\- Scripts should read input with `Input.get\_action\_strength` or `Input.is\_action\_pressed`



\## Naming conventions



\- Scenes:

&nbsp; - `Player.tscn`, `Enemy\_Slug.tscn`, `Room\_Base.tscn`, `Room\_Boss.tscn`

\- Scripts:

&nbsp; - Same base name as the main scene when possible:

&nbsp;   - `Player.gd` for `Player.tscn`

&nbsp;   - `Enemy\_Slug.gd` for `Enemy\_Slug.tscn`

\- Nodes:

&nbsp; - Use PascalCase for main nodes and clear roles:

&nbsp;   - `Player`, `EnemySlug`, `RoomRoot`, `Tiles`, `Entities`



\## Relationship to the main spec



These notes complement:



\- `docs/godot\_topdown\_spec.md`  

&nbsp; Runtime rules there are the hard constraints  

&nbsp; These notes explain \*\*how\*\* to implement those constraints in Godot scenes and scripts.



