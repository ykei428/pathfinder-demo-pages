\# PROJECT OVERVIEW



Purpose: This document is a living technical spec for the current prototype.  

It is meant to guide both humans and AI tools (Copilot, MCP, etc) when implementing code, assets and systems, and should be treated as the main source of truth for constraints, style and current decisions.



Status: This is an early prototype spec.  

Nothing is fully locked. The goal is to test a 2.5D mix of 3D trim sheet environments and 2D Spine characters.  

If this approach feels bad or too heavy, the project may fall back to a full 2D pipeline later.  

AI tools are allowed to question or suggest changes to this spec if something seems technically wrong, inconsistent or clearly harmful to the goals.



\- Top-down 2.5D dungeon game

\- Visual inspiration: The Binding of Isaac and Morsels

\- Engine: Godot 4.6.3

\- Target platform: Steam / PC  

&nbsp; - Secondary: Broad mobile support is a bonus but low priority





\## RUNTIME \& CONSTRAINTS (GODOT 4.6)



\### Engine and language



\- Godot 4.6.3

\- GDScript as primary scripting language



\### World rules



\- Gameplay on a flat XZ plane

\- No stairs, plateaus or vertical traversal

\- Tile size: 2×2 meters

\- Rooms aligned to a 2×2 meter tile grid



\### Camera



\- Camera is locked top down with slight perspective (close to isometric)

\- Camera is stationary with no tilt or rotation. A slight zoom may be used



\### Lighting constraints


\- Environment lighting and ambient occlusion baked directly into diffuse trim sheets and props

\- Limited number of non shadow casting lights:

&nbsp; - 1 ambient or directional light to remove pure black

&nbsp; - 1 point light following the player like a torch

&nbsp; - 1 spotlight per room

&nbsp; - Up to 2 extra small lights if needed



\### Physics and simulation



\- No full physics simulation needed

\- Gravity disabled for gameplay objects

\- Players and enemies move in X and Y only, no staircases or vertical up or down movement



\### Character count per scene



\- 1 player controlled character

\- Up to 20 simpler enemy characters, this is soft limit and not set in stone

\- 1 boss character in special rooms





\## WORLD \& TILESET



\### Modular room construction



\- Dungeon rooms built from modular instanced prefabs

\- Prefabs use 2×2 meter tiles for both walls and floors

\- Prefabs are instanced for consistency and performance



\### Trim sheets and meshes



\- Wall trim sheet:

&nbsp; - 1 trim sheet texture dedicated to walls

&nbsp; - Mesh: 2×2 meter vertical plane, thin and one sided

&nbsp; - Snaps to the 2×2 meter world grid



\- Floor and door trim sheet:

&nbsp; - 1 trim sheet texture that includes floor tile trims and door trims

&nbsp; - Floor mesh: 2×2 meter horizontal plane using the floor region of the trim

&nbsp; - Door mesh: unique low poly mesh using the door region of the trim

&nbsp; - Door footprint fits the 2×2 meter grid so the generator can treat it as a tile



\- General rules:

&nbsp; - Tiles are thin, mostly one sided, set up to snap edge to edge on a 2×2 meter grid

&nbsp; - UVs are aligned so each tile fits cleanly into its trim sheet region

&nbsp; - Lighting and AO are baked into the diffuse textures, no Godot lightmap baking



\### Room structure and shapes



\- Rooms built from a boolean tile grid:

&nbsp; - `true` = floor tile present

&nbsp; - `false` = empty space

\- Rooms may start as simple rectangles then expand to:

&nbsp; - Alcoves

&nbsp; - Side pockets

&nbsp; - L shapes

&nbsp; - Irregular silhouettes





\## CHARACTERS \& ANIMATION



\### Spine details



\- Using Spine Enterprise

\- Characters authored as 2D skeletal rigs

\- Placed visually in a 3D environment as billboards or sprite planes



\### Player character animations (30  fps)



\- Side facing walk cycle:

&nbsp; - Unique left walk and right walk animations (no horizontal flipping)

\- Front facing walk cycle:

&nbsp; - Used for moving down

\- Back facing walk cycle:

&nbsp; - Used for moving up

\- Attack  
&nbsp; - Side facing: Unique left attack and right attack animations (no horizontal flipping)  
&nbsp; - Front facing: Used for attacking downwards  
&nbsp; - Down facing: Used for attacking upwards  

\- Getting hit  
&nbsp; - Side facing: Flipped horizontally for left and right  
&nbsp; - Front facing: Used for getting hit downwards  
&nbsp; - Down facing: Used for getting hit upwards  

\- Dodge  
&nbsp; - Side facing: Flipped horizontally for left and right  
&nbsp; - Front facing: Used for dodging downwards  
&nbsp; - Down facing: Used for dodging upwards  

\- Die and terminate  
&nbsp; - Side facing only



\### Enemy character animations (25 fps)



\- Attack  
&nbsp; - Side facing: Unique left attack and right attack animations (no horizontal flipping)  
&nbsp; - Front facing: Used for attacking downwards  
&nbsp; - Down facing: Used for attacking upwards  

\- Getting hit  
&nbsp; - Side facing: Flipped horizontally for left and right  
&nbsp; - Front facing: Used for getting hit downwards  
&nbsp; - Down facing: Used for getting hit upwards  

\- Die and terminate  
&nbsp; - Side facing only



\### Use in 3D, this needs to be testes since the Spine plugin does not support true 3D, and requires a workaround,



\- Characters are visually 2D but exist and move in a 3D scene

\- Supported approaches:

&nbsp; - Use Spine runtime and render output onto a plane or `Sprite3D`

&nbsp; - Export sprite sheets from Spine and use:

&nbsp;   - `AnimatedSprite3D`

&nbsp;   - or a quad mesh with UV animated via `AnimationPlayer`

\- Current leaning:

&nbsp; - Use Spine runtime for the more polished player and boss characters

&nbsp; - Use sprite sheets for simpler enemies



\### Direction logic (player)



\- Up movement:

&nbsp; - Use back facing walk animation

\- Down movement:

&nbsp; - Use front facing walk animation

\- Left and right movement:

&nbsp; - Use side facing walk animation

&nbsp; - Flip horizontally for left and right

\- The same directional pattern can be reused for variants of attack, getting hit and dodge



\### Player stats



\- HP (energy bar)

\- Speed (movement speed)

\- Defense (armor amount)

\- Attack (damage dealt)

\- Luck (optional, higher luck increases drop rate of rare items)

> **Implementation:** v1 uses seven primary stats (Vitality, Intelligence, Strength, Agility, Dexterity, Defense, Luck). See [character_stats.md](../character_stats.md) and [ai/deep-research-stats.md](../ai/deep-research-stats.md).



\### Enemy stats



\- HP

\- Speed (higher value moves faster)

\- Defense

\- Attack





\## LIGHTING \& SHADING



\### Environment



\- Environment lighting:

&nbsp; - Soft vertical light and AO baked directly into diffuse textures for:

&nbsp;   - Trim sheets

&nbsp;   - Props such as chests and loops

\- Non shadow casting dynamic lights:

&nbsp; - Ambient or directional light for global fill

&nbsp; - One point light following the player (torch feel)

&nbsp; - One spotlight per room

&nbsp; - Optional extra lights when safe for performance

\- Static shadows can be faked with flat 2D shadow sprites slightly above the floor under props and characters



\### Characters



\- Character rendering:

&nbsp; - Characters rendered as `Sprite3D` or `AnimatedSprite3D` in the 3D scene

&nbsp; - Two main shading modes to experiment with:

&nbsp;   - Simple shading so 3D lights tint the sprites

&nbsp;   - Mostly flat or unlit shading to match the baked look of the environment



\- Dynamic effects:

&nbsp; - Pulsing and flickering can be faked by:

&nbsp;   - Modulating sprite color or emission over time

&nbsp;   - Adjusting intensity or color of the following point light



\- Rim light wishlist:

&nbsp; - Add fake overlay rim lighting on player and possibly enemies

&nbsp; - Direction dependent variants:

&nbsp;   - Rim overlay that mirrors with left and right side walk

&nbsp;   - Different rim overlay for up direction

&nbsp; - Target style:

&nbsp;   - Slight warm edge light from top right

&nbsp;   - Subtle blue rim from bottom left



\- Character count context:

&nbsp; - 1 player

&nbsp; - Up to 5 enemies

&nbsp; - 1 boss in special rooms

&nbsp; - Mixed use of Spine runtime for hero and bosses and sprite sheets for enemies is allowed





\## PHYSICS \& COLLISION



\### General



\- No full physics simulation

\- No vertical traversal

\- Game logic is effectively 2D movement on a flat 3D plane



\### Player body



\- `CharacterBody2D` with `move_and_slide()`

\- Gravity disabled

\- Movement in 2D pixel space (X = left/right, Y = up/down)

\- All speeds/distances defined in metres, converted to pixels at `_ready()` via `Game.PIXELS_PER_METER` (PPM = 100)



\### 2D → 3D Projection



\- Gameplay runs entirely in 2D pixel space. The 3D environment is purely visual.

\- `Game.PIXELS_PER_METER = 100.0` — 1 pixel = 1 cm

\- `Game.GAMEPLAY_ANCHOR = Vector2(960, 540)` — screen centre at 1080p

\- 2D→3D: `pos_3d.x = (pos_2d.x - ANCHOR.x) / PPM`, `pos_3d.z = (pos_2d.y - ANCHOR.y) / PPM`

\- Camera FOV is cosmetic only and MUST NOT affect gameplay coordinates

\- NEVER use camera raycasting for 2D→3D projection; always use the linear PPM mapping



\### Collision



\- Player:

&nbsp; - `CollisionShape2D` (capsule) on the CharacterBody2D

\- Environment (2D gameplay):

&nbsp; - `StaticBody2D` line-segment wall blockers in pixel space, auto-generated from the boolean tile grid

&nbsp; - Edges merged for efficiency; door gaps skipped

\- Environment (3D visuals):

&nbsp; - `StaticBody3D` box colliders on 3D props

&nbsp; - These do NOT participate in 2D gameplay physics



\### Triggers and interaction



\- `Area2D` used for:

&nbsp; - Door triggers

&nbsp; - Hit detection (hurtboxes)

&nbsp; - Pickups and other interaction zones





\## PROCEDURAL ROOM GENERATION



\### Abstract room data



\- Rooms exist as data first

\- Each room has:

&nbsp; - A grid position on a high level map

&nbsp; - Flags for exits in directions N, E, S, W



\### Room graph generation



\- Start with a start room

\- Use random walk or branching to construct a graph of rooms

\- One room is marked as the final exit room

\- Adjacency between rooms defines door connections:

&nbsp; - If room A is north of room B:

&nbsp;   - A has a south door

&nbsp;   - B has a north door

\- Rules will define how far the exit can be from the start



\### Tile builder



\- For each room:

&nbsp; - Build a 2D boolean grid:

&nbsp;   - `true` = floor tile present

&nbsp;   - `false` = empty

&nbsp; - Use 2×2 meter 3D tiles driven by this grid:

&nbsp;   - Place floor tiles where grid is true

&nbsp;   - For each floor tile with an adjacent false or out of bounds cell:

&nbsp;     - Place a wall tile along that edge

&nbsp;   - At exit positions:

&nbsp;     - Replace wall tiles with door meshes

\- Room instances are placed in world space as:

&nbsp; - `room\_grid\_position \* room\_world\_size`

\- This keeps rooms snapped and non overlapping



\### Floor lifecycle and room state contract



- Floor generation timing:

&nbsp; - Generate full floor graph data before the floor starts.

&nbsp; - Instantiate playable room scene only when entered.

&nbsp; - Unload previous room scene during transitions.

- Start room and options:

&nbsp; - Start room always has 4 door options (N/E/S/W).

- Exit room policy:

&nbsp; - Exit room is generated at floor creation.

&nbsp; - Exit marker remains hidden until discovered.

&nbsp; - Entering the exit room advances to next floor.

- Room count target:

&nbsp; - Each floor targets a random room count in the 6 to 12 range.

- Room state ownership (`Dungeon.rooms`):

&nbsp; - Core fields: `type`, `doors`, `discovered`, `cleared`

&nbsp; - Runtime support fields: `spawn_mode`, `base_enemy_count_override`, `encounter_seed`, `visited_count`

- Combat spawn rules:

&nbsp; - Combat rooms lock doors on entry and unlock on clear.

&nbsp; - Non-combat rooms use `spawn_mode = none` and should not lock for combat.

&nbsp; - Cleared rooms must never respawn enemies when revisited.



\### Progression and persistence boundaries



- Floor scope (`Dungeon`):

&nbsp; - Room graph and per-room clear/discovery state for current floor only.

&nbsp; - Reset when next floor is generated.

- Run scope (`Game`):

&nbsp; - Gold and run build state persist between floors within an active run.

&nbsp; - Run inventory/items/buffs are run-scoped and cleared when run ends unless explicitly converted.

- Profile scope (`Save`):

&nbsp; - Persistent between runs: banked currency, meta XP, unlocks/upgrades.



\### Difficulty scaling and balancing knobs



- Difficulty should scale with floor level through editable data, not hardcoded per-script constants.

- Required channels:

&nbsp; - Enemy count scaling by floor (with min/max caps)

&nbsp; - Enemy stat scaling by floor (HP, damage, speed multipliers)

&nbsp; - Reward scaling by floor (gold/loot budget)

- Controlled randomness:

&nbsp; - Seeded per-floor/per-room variance for reproducibility

&nbsp; - Small bounded variance bands to avoid extreme spikes

- Balancing workflow:

&nbsp; - Keep scaling knobs in one data source under `project/data/`

&nbsp; - Preserve deterministic seed replay for balancing and bug repro

&nbsp; - Expose debug readouts of computed encounter budget/multipliers





\## TEXTURE \& ASSET PIPELINE



\### Textures



\- Diffuse maps:

&nbsp; - Baked lighting and AO directly into diffuse textures

&nbsp; - Authoring tools:

&nbsp;   - 3D Coat

&nbsp;   - Blender

&nbsp;   - Substance Painter

&nbsp; - Workflows using SoMuchMaterials or Simple Diffuse style setups

&nbsp; - Format: PNG, lossless



\- Normal maps:

&nbsp; - Standard tangent space normal maps

&nbsp; - Format: PNG, lossless



\- Packed PBR maps (optional):

&nbsp; - Channel layout:

&nbsp;   - R = Ambient Occlusion (AO)

&nbsp;   - G = Roughness

&nbsp;   - B = Metallic

&nbsp;   - A = unused

&nbsp; - Stored as PNG even if alpha is unused

&nbsp; - CRITICAL: In Godot StandardMaterial3D, set `metallic = 1.0` when assigning a metallic texture, because Godot multiplies the texture sample by the base scalar (default 0.0 = always zero)



\- Trim resolution:

&nbsp; - 1024×1024 per trim sheet is the main target

&nbsp; - Lower resolutions can be tested later for performance or memory



\### Mesh formats



\- Environment meshes exported as `.gltf` with external PNG textures

\- Reasons:

&nbsp; - Multiple prefabs share the same trim textures

&nbsp; - Avoids duplicated embedded textures in each mesh

&nbsp; - `.glb` avoided for shared trim setups



\### Mesh scenes



\- FloorTile:

&nbsp; - 2×2 meter floor plane, UV mapped into floor region of trim

\- WallTile:

&nbsp; - 2×2 meter vertical plane, UV mapped into wall region of trim

\- DoorTile or custom door mesh:

&nbsp; - Unique low poly mesh using door region of trim sheet

\- All meshes UV mapped to shared trim sheets so the generator only needs to place them on the 2×2 grid
