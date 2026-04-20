# Amritanshu Jain — Portfolio Website

Personal portfolio website for Amritanshu Jain, Mechanical Design Engineer, with interactive 3D CAD file viewer.

## Features

- **Interactive 3D CAD Viewer** — Drag, rotate, pan, and zoom your models directly in the browser
- **Supported formats:** `.STL` · `.OBJ` · `.PLY`
- **Drag & drop upload** — Add your CAD files instantly to the gallery
- **Per-project details** — Name and describe each project
- **Model appearance controls** — 6 color swatches, wireframe mode, flat shading toggle
- **Thumbnail generation** — Auto-renders a preview for each uploaded file
- **Fully static** — No backend required, works perfectly on GitHub Pages

## Deploying to GitHub Pages

1. **Create a new repo** on GitHub (e.g. `username.github.io` or any name)

2. **Upload just `index.html`** to the root of the repo

3. **Enable GitHub Pages:**
   - Go to repo → **Settings** → **Pages**
   - Under *Source*, select **Deploy from a branch**
   - Choose **main** branch and **/ (root)**
   - Click **Save**

4. Your site will be live at:
   - `https://username.github.io` (if repo is named `username.github.io`)
   - `https://username.github.io/repo-name` (for any other repo name)

## Using the CAD Viewer

1. Navigate to the **CAD Projects** section
2. Drag & drop `.stl`, `.obj`, or `.ply` files onto the upload zone (or click to browse)
3. A card with a thumbnail preview appears in the gallery
4. Click **View 3D →** on any card to open the full interactive viewer
5. Use mouse to explore:
   - **Left drag** → Rotate
   - **Right drag** → Pan
   - **Scroll** → Zoom
   - **Double click** → Reset view

> **Note:** Uploaded files are processed entirely in your browser — nothing is sent to any server.

## Tech Stack

- Pure HTML / CSS / JavaScript (zero dependencies, zero build step)
- [Three.js r128](https://threejs.org/) via CDN for 3D rendering
- Google Fonts (Bebas Neue, DM Mono, Outfit)

## Customization

All content is in `index.html`. To update:
- **Your info** — Search for the section comments (`<!-- Hero -->`, `<!-- Experience -->`, etc.)
- **Sample project cards** — Edit the placeholder cards in `#projectsGrid`
- **Colors** — CSS variables at the top of `<style>` (`:root { ... }`)
