\# Spine Runtime Notes



Purpose: Document how Spine is used in this project (editor setup, exports, runtimes and naming), so both humans and AI tools treat Spine assets consistently.



\## Spine setup



\- License: \*\*Spine Enterprise\*\*

\- Role: Create 2D skeletal animations for:

&nbsp; - Player character

&nbsp; - Boss characters

&nbsp; - Simpler enemies

\- Integration target: Godot based \*\*2.5D topdown\*\* dungeon game



\## Runtime vs sprite sheets



Current plan for this prototype:



\- Player and bosses:

&nbsp; - Prefer \*\*Spine runtime\*\* or higher fidelity pipeline

&nbsp; - Richer animation control and possible future features

\- Enemies:

&nbsp; - Prefer \*\*sprite sheet exports\*\* from Spine

&nbsp; - Used as `AnimatedSprite3D` or simple textured quads in Godot

\- This mix can change later if performance or workflow suggests a different split



\## Animation settings



\- Target framerate in Spine for this project: \*\*25 FPS\*\*  

&nbsp; (matches exported frame rate for sprite sheets)

\- Key animation names for the player:

&nbsp; - `walk\_side`

&nbsp; - `walk\_back`

&nbsp; - `attack`

&nbsp; - `hit`

&nbsp; - `dodge`

&nbsp; - `die`

\- Key animation names for enemies:

&nbsp; - `attack`

&nbsp; - `hit`

&nbsp; - `die`



\## Direction logic



\- Side facing walk is used for:

&nbsp; - Moving left and right (mirror horizontally)

&nbsp; - Moving down (reuse side view)

\- Back facing walk is used for:

&nbsp; - Moving up

\- Same pattern can be applied to attacks and dodges if directional variants are added.



\## Export workflow



\### For Spine runtime usage



\- Export types:

&nbsp; - Spine skeleton data as JSON or binary

&nbsp; - Texture atlas for the skeleton

\- Data use:

&nbsp; - Loaded by the Spine runtime for Godot (or via a custom integration)

&nbsp; - Animations are controlled from Godot scripts:

&nbsp;   - Set current animation

&nbsp;   - Queue transitions

&nbsp;   - Play hit, attack, dodge, die



\### For sprite sheet usage



\- Export type:

&nbsp; - Animated sprite sheets at \*\*25 FPS\*\*

\- Suggested naming:

&nbsp; - `player\_walk\_side\_strip.png`

&nbsp; - `player\_walk\_back\_strip.png`

&nbsp; - `enemy\_slug\_walk\_strip.png`

&nbsp; - `enemy\_slug\_attack\_strip.png`

\- Frame order:

&nbsp; - Left to right for each animation strip

\- Godot usage:

&nbsp; - Use `AnimatedSprite3D` with sprite frames defined from the exported sheets

&nbsp; - Or use a quad with UV animation driven by `AnimationPlayer`



\## Folder and naming conventions



\- Suggested folder layout in the repo:



&nbsp; - `art/spine/player/`

&nbsp;   - Spine project file: `player.spine` or `player.spine-project`

&nbsp;   - Exports:

&nbsp;     - `player\_skeleton.json` or `.skel`

&nbsp;     - `player\_atlas.atlas`

&nbsp;     - `player\_atlas.png`

&nbsp;     - Sprite sheets if used: `player\_walk\_side\_strip.png`, etc

&nbsp; - `art/spine/enemies/Enemy\_Slug/`

&nbsp;   - `enemy\_slug.spine`

&nbsp;   - `enemy\_slug\_skeleton.json` or `.skel`

&nbsp;   - `enemy\_slug\_atlas.\*`

&nbsp;   - Sprite sheet exports



\- Skeleton names:

&nbsp; - `player`

&nbsp; - `enemy\_slug`

&nbsp; - `boss\_knight` (example)

\- Consistent animation names across skeletons when possible:

&nbsp; - `walk\_side`, `walk\_back`, `attack`, `hit`, `dodge`, `die`



\## Skins and equipment



\- Skins can be used for:

&nbsp; - Weapon variants (different swords, staffs, guns)

&nbsp; - Cosmetic variations (color swaps, armor variants)

\- Weapon ideas:

&nbsp; - Put weapons in separate slots that can be swapped by changing skin, attachment or mix of both

\- This spec does not lock down a final skin strategy, but:

&nbsp; - The Spine rigs should be set up so adding skins later is possible without redoing the skeleton



\## Relationship to the main spec



These notes complement:



\- `docs/godot\_topdown\_spec.md`

\- `docs/godot\_runtime\_notes.md`



The main spec defines what animations and behavior are needed in game.  

These Spine notes define how that is organized, named and exported from Spine for Godot.



