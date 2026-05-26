# Pathfinder public web demo (GitHub Pages)

Static Godot web exports live under **`docs/`** (desktop) and **`docs/mobile/`** (touch controls).

## GitHub Pages setup (required once)

**Recommended:** deploy from the branch — no Actions build step.

1. Repo **Settings → Pages**
2. **Build and deployment → Source:** `Deploy from a branch`
3. **Branch:** `main` · **Folder:** `/docs`
4. Save, wait 2–5 minutes

Then:

| URL | Build |
|-----|--------|
| https://ykei428.github.io/pathfinder-demo-pages/ | `docs/` (keyboard/mouse) |
| https://ykei428.github.io/pathfinder-demo-pages/mobile/ | `docs/mobile/` (touch) |

Check versions:

- https://ykei428.github.io/pathfinder-demo-pages/DEMO_VERSION.txt
- https://ykei428.github.io/pathfinder-demo-pages/mobile/DEMO_VERSION.txt

## Alternative: GitHub Actions

Workflow [`.github/workflows/pages.yml`](.github/workflows/pages.yml) uploads the `docs/` folder when Actions is the Pages source. If the default **pages build and deployment** workflow fails, disable it or switch to **branch `/docs`** above.

Published from the private `pathfinder` repo via `publish-demo` / `publish-demo-mobile`.
