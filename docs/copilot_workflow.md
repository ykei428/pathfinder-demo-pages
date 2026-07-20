\# Copilot workflow cheat sheet (Pathfinder / Godot 4.6.3)



This file summarizes how I want to use GitHub Copilot and Copilot Chat in this project.



---



\## 1. When to use inline vs Copilot Chat



Inline = the grey “ghost text” suggestions that appear directly in the editor while I type.



Use inline for:

\- Completing a function I already started

\- Filling in loops, conditions and small bits of logic

\- Expanding a short comment into a function



Example:

\# Return a random empty tile from the grid

func get\_random\_empty\_tile(grid: Array) -> Vector2:

&nbsp;   # let Copilot finish this



Copilot Chat = the separate chat panel.



Use Copilot Chat for:

\- Generating whole scripts or big chunks (Player.gd, RoomGenerator.gd)

\- Refactoring or cleaning up messy code

\- Explaining code, debugging and planning structure



Example prompts:

\- /explain #selection

\- /refactor #selection to follow Godot 4.6.3 typed GDScript

\- Using docs/godot\_topdown\_spec.md, create a basic RoomGenerator.gd



---



\## 2. Helpful tags in code



Use comment tags to mark areas and steer Copilot later:



\- #todo     = something I plan to add

\- #fixme    = known bug or hack

\- #refactor = needs structural cleanup

\- #optimize = improve performance or allocation

\- #explain  = ask Chat to explain this later



Example:

\#todo add more enemy patterns once base movement works



---



\## 3. Useful VS Code extensions for this project



\- GitHub Copilot

\- GitHub Copilot Chat

\- Godot Tools (GDScript language server and scene integration)



\## 4. Prompt patterns I want to reuse



Spec aware prompts:

\- Using docs/godot\_topdown\_spec.md, write CharacterBody3D movement with move\_and\_slide.

\- Using docs/godot\_topdown\_spec.md and docs/godot\_runtime\_notes.md, create RoomGenerator.gd that builds a 2x2 meter tile room from a boolean grid.

\- Using docs/spine\_runtime\_notes.md, suggest animation names and export settings for a new enemy.



Debug and refactor:

\- /explain #selection

\- /refactor #selection to use Godot 4.6.3 style and typed GDScript.

\- /optimize #selection for fewer allocations per frame.



Small focused request:

\- Suggest a cleaner way to handle directional input for the player using InputMap actions.



---



\## 5. Tips for better GDScript suggestions



\- Use typed GDScript:

&nbsp; var velocity: Vector3 = Vector3.ZERO



\- Always extend the correct node:

&nbsp; extends CharacterBody3D

&nbsp; extends Node3D

&nbsp; extends Node



\- Use clear names:

&nbsp; spawn\_enemies(), generate\_room(), get\_exit\_positions()



\- Keep related scripts open in VS Code while asking Copilot.

\- Start scripts with a short comment describing their role:

Save system notes for future chats:
- Use `autoload/save.gd` as the single save authority.
- Treat save slots as generic player profiles (default 3).
- Keep manual save between runs only; do not add mid-run manual save.
- Keep autosave on run-end events.

&nbsp; # Handles procedural dungeon room generation for this project

---

\## 6. Chat shortcuts used in this repo

- `start MCP` *(optional — not required for Cursor sessions)*
	Runs `& ".\\mcp_local\\start-server.ps1"` in PowerShell.
	Starts the local REST search API (port 4000) and tries to open `project/project.godot`.
	Agents still verify code via `run-godot-headless.ps1` and other `mcp_local/` scripts without this server.

- `push now`
	Treated as explicit approval for release flow in chat.
	It should bump `project/project.godot` `config/version`, stage changes, commit with the required version subject format, and push.

- `publish demo`
	Runs `mcp_local/publish-demo.bat`.
	This exports a Godot Web build to `docs/index.html`, validates expected web artifacts, syncs to the configured public demo repo, commits, and pushes.
	For workaround smoke test only: add `-SkipExport -AllowMissingWebArtifacts -DemoRepoPath "<path>"`.

- `create demo`
	Runs `mcp_local/create-demo.bat`.
	Exports a versioned Windows x64 desktop demo to `dist/Pathfinder-vX.Y.Z/` and zips it for local sharing.



