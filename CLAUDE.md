# amritanshujain.com — Project Context

## Project
Personal portfolio for Amritanshu Jain, Lead Hardware Engineer / Founding Engineer at KinesthetIQ Robotics Studio, Bengaluru. Hosted on GitHub Pages at `amritanshu-j/amritanshujain`, custom domain `amritanshujain.com` (also owns `.in`, redirects to `.com`).

---

## Tech Stack
Single `index.html` — no frameworks, no build step. All CSS and JS inline.

**CDN dependencies:**
- Google Fonts: Bebas Neue, DM Mono, Outfit
- Three.js r128 (Cloudflare CDN)
- STLLoader r128 (jsDelivr)
- OrbitControls r128 (jsDelivr)

---

## File Structure
```
index.html
sitemap.xml
robots.txt
CNAME
CLAUDE.md
images/
  arm-blueprint.svg
  ARM ON QUADRUPED.webp
  kinesthetiq-logo.svg
  og-preview.jpg          ← headshot, 413×531px, used as OG/Twitter/JSON-LD image
  favicon.svg
  landing-page-photo.webp ← 961KB, no longer used on main site
  og-preview.jpg          ← headshot, 413×531px — OG/Twitter/JSON-LD image AND About section identity card photo
  favicon-192.png         ← 192×192 PNG, also used as apple-touch-icon
  favicon-48.png          ← 48×48 PNG
  favicon-32.png          ← 32×32 PNG
URDF/
  urdf_manipulator_v2/    ← full ROS package exported from SolidWorks
    urdf/
      urdf_manipulator_v2.SLDASM.urdf
      urdf_manipulator_v2.SLDASM.csv
    meshes/               ← STL file sizes (total ~40MB — check before Cloudflare migration)
      base_link.STL       2.94 MB
      p1_manipulator.STL  7.67 MB
      p2_manipulator.STL  11.29 MB
      p3_manipulator.STL  7.26 MB
      r1_manipulator.STL  0.02 MB
      y1_manipulator.STL  3.75 MB
      y2_manipulator.STL  7.27 MB
    config/
    launch/
      display.launch
      gazebo.launch
    CMakeLists.txt
    package.xml
    export.log
    joint_names_urdf_manipulator_v2.SLDASM.yaml
urdf-viewer/
  index.html              ← standalone drag-drop URDF tool (separate page, see below)
```

---

## Design Tokens
```css
/* Dark (default) */
--bg: #080b10       --bg2: #0d1117      --bg3: #111820
--line: #1e2d3d     --steel: #8fb4cc    --accent: #00d4ff
--accent2: #ff6b35  --accent3: #8b7cf6  --text: #c9d8e8
--text-dim: #5a7a94 --white: #eef4f9    --grid-col: rgba(0,212,255,0.12)

/* Light overrides (html[data-theme="light"]) */
--bg: #f0f4f8       --bg2: #e6edf5      --bg3: #dce5ef
--line: #b8cad8     --steel: #3d6080    --accent: #007aaa
--accent2: #e05515  --accent3: #5b4fd8  --text: #1a2a3c
--text-dim: #4a6a84 --white: #0a1520    --grid-col: rgba(0,122,170,0.07)
```

Three-accent system: cyan (primary), orange (secondary), violet (`--accent3`, added 2026-06-30). Applied as a colour spectrum across the UI — per-card timeline dots, per-section header lines, per-skill-block titles, per-contact-link hovers.

---

## Site Sections (scroll order)
- **Loader** — planetary gearbox splash screen (canvas `#gearCanvas` 340×340, 8:1 reduction animation, hides **2200ms** after `window.load`, cancels RAF). `.loader-glass` frosted card wraps the name — see Loader section below.
- **Nav** — fixed, liquid glass, logo animates to full name on scroll, Blog slide-in panel. Structure: `.nav-logo` (left) + `.nav-links` (desktop links + blog toggle) + `.nav-actions` (theme toggle + hamburger — always visible). Mobile hamburger (≤640px) opens `.mobile-menu` slide-down panel.
- **Hero** — tagline, name (Bebas Neue), role, CTAs (View Work / Get In Touch / Resume ↗), KinesthetIQ logo, Three.js URDF viewer
- **Experience (01)** — `.xp-card` per role, Photos/CAD/Videos tabs, carousel with progress bar + counter, lightbox (images + Drive video iframes)
- **Skills (02)** — 4 blocks, each with a different accent colour title (cyan/orange/violet/steel)
- **About (03)** — identity card (photo + bio) + credentials row (CSWP + B.Tech) + awards list. See About section below.
- **Contact** — phone (cyan hover), email (orange hover), LinkedIn (violet hover), GitHub (steel hover)
- **Footer** — `© 2026 AMRITANSHU JAIN` (left) · `BENGALURU, KARNATAKA · INDIA` (right)

---

## URDF Viewer — Hero (index.html)

### Renderer
```js
WebGLRenderer({ canvas, alpha: true, antialias: true })
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
renderer.shadowMap.enabled = true
```

### Canvas / Layout (desktop)
- Container: `.bp-layer` — `position:absolute; top:calc(50% + 16rem); right:-2%; transform:translateY(-50%); width:68%; height:125%; z-index:0; overflow:visible;`
- `#urdfControls` — `position:fixed; bottom:6rem; left:70%; z-index:2;` — scrollable with `max-height:calc(100vh - 6rem)`

### Canvas / Layout (mobile ≤768px)
```css
.bp-layer {
  display: block;
  position: absolute;
  inset: 0;
  width: 100%; height: 100%;
  transform: none;
  opacity: 0.45;
  z-index: 0;
  pointer-events: none;
  overflow: hidden;
}
```
- `#urdfControls` hidden (`display:none!important`)
- Camera auto-scaled to fit arm in frame based on canvas aspect ratio
- Camera X offset `dist*0.26` shifts arm left
- Small phones (≤480px): opacity 0.37

### Camera (desktop)
```js
camera = new THREE.PerspectiveCamera(40, 1, 0.0001, 10000);
camera.position.set(md*0.3, md*0.5, md*3.5);
controls.target.set(0, 0, 0);
```

### Camera (mobile — computed in resize())
```js
const tanHalfVFov = Math.tan(camera.fov/2 * Math.PI/180);
const tanHalfHFov = tanHalfVFov * (w/h);
const dist = robotMd * 0.8 / Math.min(tanHalfVFov, tanHalfHFov);
camera.position.set(dist*0.26, robotMd*0.05, dist);
// robotMd is let-hoisted in IIFE scope; resize() checks `if(robotMd && w<=768)`
```

### Controls (OrbitControls — fully locked)
```js
controls.enableRotate = false;
controls.enableZoom   = false;
controls.enablePan    = false;
```
No orbit, pan, or zoom. Only joint sliders move the arm.

### Robot Orientation
```js
robotGroup.rotation.set(Math.PI*0.5, Math.PI*1, Math.PI*0.1);
```

### Material
```js
MeshStandardMaterial({
  color: 0xb2bcc8, metalness: 0.72, roughness: 0.28,
  envMapIntensity: 1.0, side: THREE.DoubleSide
})
```

### Lighting (4-point rig)
```js
AmbientLight(0xdde8f0, 0.25)                          // soft cool ambient
DirectionalLight(0xffffff, 1.4) @ (6, 10, 7)          // key
DirectionalLight(0x8aaabb, 0.45) @ (-5, 2, -4)        // fill
DirectionalLight(0x00d4ff, 0.35) @ (-3, 4, -8)        // cyan rim (matches accent)
DirectionalLight(0x334455, 0.2)  @ (0, -6, 0)         // under-fill
```

### Joint Limits (set in JS via limitMap, match these in the URDF too)
| Joint   | Lower  | Upper |
|---------|--------|-------|
| yaw 1   | -3.14  | 3.14  |
| pitch 1 | -3.14  | 0.30  |
| pitch 2 |  0     | 3.14  |
| pitch 3 | -1     | 2.25  |
| yaw 2   | -1.4   | 1.4   |
| roll 1  | -3.14  | 3.14  |

### Intro Animation (plays once on load, then sliders take over)
Keyframes using actual radian values:
| Time  | yaw1  | pitch1 | pitch2 | pitch3 | yaw2  | roll1 |
|-------|-------|--------|--------|--------|-------|-------|
| 0s    | 0     | 0      | 0      | 0      | 0     | 0     |
| 2.0s  | 1.0   | -0.5   | 0      | 0      | 0     | 0     |
| 3.5s  | 1.0   | -0.5   | 1.5    | 1.0    | 0.5   | 0     |
| 5.0s  | 1.0   | -0.5   | 1.5    | 1.0    | 0.5   | 2.0   |
| 6.5s  | -1.0  | -0.8   | 0.8    | -0.5   | -0.5  | -2.0  |
| 8.5s  | 0     | 0      | 0      | 0      | 0     | 0     |

Uses `easeInOut` interpolation. After final frame, `introAnim = null` and sliders take over.

### Render Pause
Rendering pauses when `#hero` scrolls out of view (IntersectionObserver, threshold 0.1) to save GPU — resumes on re-entry.

---

## Loader — Planetary Gearbox Splash Screen

Canvas `#gearCanvas` (340×340 px) animates a planetary gearbox while the page loads. Hidden **2200ms** after `window.load` fires (`cancelAnimationFrame` + `.hidden` class → `opacity:0; visibility:hidden`).

```
Sun gear:   Z=12, ω = 1.4 rad/s (input)
Planet:     Z=36, 3 planets at 120° spacing, carrier ω = Wsun/8
Ring:       Z=84, fixed
Ratio:      8:1 (displayed as "8 : 1" text on canvas)
```
- Colors: CYAN `#00d4ff` on BG `#060d18`
- Progress bar at bottom animates continuously (does NOT track real load progress)

### Glass card
`.loader-glass` wraps **only** the name (no subtitle). Sequence:
- `0.4s` → glass card fades in (`lnFade 1s ease-out`)
- `0.72s` → "AMRITANSHU JAIN" fades in inside card (`lnFade 1.1s ease-out`)
- `2.2s` → whole loader fades out

Glass card style: `rgba(255,255,255,.04)` bg, `rgba(255,255,255,.09)` border, `blur(18px) saturate(1.4)` backdrop, `inset 0 1px 0 rgba(255,255,255,.13)` top-rim highlight, `0 0 60px rgba(0,180,255,.06)` ambient cyan glow. Light-mode override: dark-tinted with white rim.

---

## Standalone URDF Viewer Tool — urdf-viewer/index.html

Separate page at `https://amritanshujain.com/urdf-viewer/`. Linked from nav via `.urdf-link` button. Full-screen tool for inspecting any URDF.

### SEO (urdf-viewer page)
- Title: "URDF Viewer Online — Free 3D Robot Model Visualizer"
- `canonical`: `https://amritanshujain.com/urdf-viewer/`
- 10 keywords targeting `urdf viewer online`, `ROS robot viewer`, etc.
- Twitter card: `summary_large_image` (unlike main page which uses `summary`)
- JSON-LD: `SoftwareApplication` schema — name, url, description, featureList, author
- OG image: same `og-preview.jpg` as main site

### Tech
- Three.js **r155** (ES module via importmap, newer than r128 used in hero)
- **Self-contained URDF parser** (no external library) — same approach as hero viewer but as ES module; replaced broken `urdf-loader@0.12.3/src/URDFLoader.js` import
- OrbitControls + STLLoader from Three.js r155 addons

### Layout
Fixed nav (64px) + side panel (300px wide, `var(--panel-w)`) + 3D viewport (flex:1). `html,body { overflow:hidden }` — no scroll.

### Camera & Controls
```js
PerspectiveCamera(45, 1, 0.001, 1000) at (2, 1.5, 3)
OrbitControls — enableDamping: true, dampingFactor: 0.07
// Free rotate / zoom / pan (opposite of locked hero viewer)
```

### Lighting
```js
AmbientLight(0xdde8f0, 0.45)           // brighter ambient than hero (0.25→0.45)
DirectionalLight(0xffffff, 1.4) @ (6, 10, 7)   castShadow: true
DirectionalLight(0x8aaabb, 0.45) @ (-5, 2, -4)
DirectionalLight(0x00d4ff, 0.30) @ (-3, 4, -8) // slightly dimmer rim (0.35→0.30)
DirectionalLight(0x334455, 0.20) @ (0, -6, 0)
ShadowMaterial floor (opacity: 0.12) — receives shadows, invisible otherwise
```

### Material
```js
// Visual meshes
MeshStandardMaterial({ color: 0xb2bcc8, metalness: 0.65, roughness: 0.3, side: DoubleSide })
// Collision meshes
MeshStandardMaterial({ color: 0xff6b35, transparent: true, opacity: 0.35, depthWrite: false })
```

### Features
- **Drag-drop** a URDF package folder (walks directory entries via `webkitGetAsEntry`; builds filename→objectURL map to resolve `package://` URIs)
- **File input** fallback (uses `webkitRelativePath`)
- **Load Sample Robot** button — loads portfolio's own `URDF/urdf_manipulator_v2/` arm with same orientation as hero (`rotation.set(Math.PI*.5, Math.PI, Math.PI*.1)`)
- Supports STL meshes; unknown formats (DAE, OBJ) load as empty `THREE.Group` with console.warn
- **Robot info panel** — link count, joint count, movable joint count
- **Options toggles**: Animate Joints, Show Visual (on by default), Show Collision, Show Grid, Show Axes
- **Animate Joints** — sinusoidal sweep: `sin(animClock*0.7 + i*0.4)` (where `i` is the movable joint index) mapped to joint limits; pauses when user touches a slider
- **Reset Camera** button
- **Joint sliders** — one per movable joint, uses `j.setJointValue(v)` from urdf-loader
- Camera framing: `dist = md*2.5`, position `(dist*.55, dist*.45, dist)` for drop-in robots; `dist = md*3`, position `(dist*.35, dist*.45, dist)` for sample robot
- Grid: `GridHelper(10, 20)` scaled to `md*0.4`; Axes: `AxesHelper(0.5)` scaled to `md*0.12`
- Light/dark theme via localStorage (same key `theme` as portfolio)

---

## Experience Cards

Five `.xp-card` entries in scroll order. Each card has `.xp-head` (date / role / company) and `.xp-body` (`.xp-pts` bullet list | `.xp-media` with tabs + carousel). Carousel auto-advances every 4200ms; pauses on hover; clicking viewer opens lightbox. Navigation uses a **progress bar** (`.xp-prog` / `.xp-prog-bar`) + slide counter (`.xp-counter`), not dots.

**Card body layout:** `grid-template-columns:3fr 2fr` (text wider, media narrower — keeps viewer shorter without distorting aspect ratio). `align-items:start` so bullet column doesn't stretch. Column divider is a `::before` pseudo-element on `.xp-body` (`left:60%; top:0; bottom:0; width:1px`) — hidden on mobile (`display:none` in 900px breakpoint). `.xp-pts` has no `border-right` (divider is on parent).

**Card meta text sizes:** `.xp-date` `.78rem`, `.xp-role` `1.2rem`, `.xp-co` `.78rem`.

**Mars Rover career progression stepper:** replaces single `.xp-role` text. Three `.career-step` divs inside `.career-track`, each with `.career-dot` + `.career-info` (`.career-role` + `.career-dates`). Steps connected by `::after` pseudo-element lines; second line uses orange gradient. Active step (Mechanical Head) has glowing orange dot. On mobile ≤480px flips to vertical column layout.

Videos tab: carousel pauses auto-advance; clicking thumbnail opens a Drive `/preview` iframe in the lightbox (lazy-loaded, zero cost until clicked — facade pattern).

Each card has a unique **timeline dot colour** (dot `::before`, hover border, date label all match):
- Card 1 KinesthetIQ: cyan (`--accent`)
- Card 2 Chirathe: orange (`--accent2`)
- Card 3 IISc: violet (`--accent3`)
- Card 4 Curiouz: steel (`--steel`)
- Card 5 Mars Rover: orange (`--accent2`)

| # | Period | Role | Company | Media |
|---|--------|------|---------|-------|
| 1 | May 2024 – Present | Lead Hardware Engineer · Founding Engineer | KinesthetIQ Robotics Studio · Bengaluru | Photos (14 imgs) + CAD/Renders (5 imgs: Totem Series + YAM Manipulator) + Videos (4 Drive videos) |
| 2 | Aug 2023 – Feb 2024 | Mechanical Design Engineer | ARTPARK · Chirathe Robotics (Strider Robotics) · Bengaluru | Photos (2 imgs) + CAD (3 imgs) |
| 3 | Dec 2022 – Jul 2023 | Research Intern | Robert Bosch Centre for Cyber-Physical Systems · IISc · Bengaluru | Photos (2 imgs) + CAD (3 imgs) |
| 4 | May 2022 – Dec 2022 | Research Intern | Curiouz TechLabs · Udupi | Photos (1 img) + CAD (1 img) |
| 5 | Nov 2020 – Sep 2023 | Mechanical Head (via stepper: Trainee → Engineer → Mechanical Head) | Mars Rover Manipal · MIT Udupi | YouTube embed only (`LtKkaTUwOCQ`, starts 38s) — no tabs |

KinesthetIQ Videos tab Drive IDs:
- `14Xfo7DfvyLyX7kA6u928vp3B0vChee10` — Autonomous Cloth Folding
- `1o1SYutSN9im2bmUsItSDtLgedqT9uBAT` — Autonomous Box Closing
- `1QhiwOfpyzlKGIB1_C5ZTFg-l5c7tJJTb` — Telescopic Teleoperation
- `1lSTppdl7f7S8J7_nATeEAi5okBtPJanO` — Perioperation Live Demo

All images served via Google Drive thumbnails (`sz=w800` carousel, `sz=w1200` lightbox). Mars Rover card has no `.xp-tabs` wrapper. The timeline vertical line is 2px wide with a blurred `::after` glow pseudo (cyan→violet→transparent).

---

## SEO — Current State
All of the following are implemented (fully live on amritanshujain.com as of 2026-06-27):
- `meta description`, `canonical`, `title` ✓
- `meta robots` — index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1 ✓
- `meta author` — Amritanshu Jain ✓
- `meta keywords` — 14 targeted keywords ✓
- `meta theme-color` — #00d4ff ✓
- Geo tags — `geo.region` (IN-KA), `geo.placename` (Bengaluru), `geo.position`, `ICBM` ✓
- Open Graph — title, description, type, url, site_name, locale, image (`og-preview.jpg` 413×531px headshot), image:width, image:height, image:alt ✓
- Twitter/X Card — card (summary), title, description, image (`og-preview.jpg`), image:alt, site (@amritj2002), creator (@amritj2002) ✓
- JSON-LD Person schema — name, url, image (`og-preview.jpg` 413×531), jobTitle, description, worksFor (KinesthetIQ), alumniOf (MIT Manipal), address (Bengaluru, Karnataka, IN), sameAs (LinkedIn + GitHub), knowsAbout (10 topics), award (4 results), hasCredential (CSWP) ✓
- JSON-LD WebSite schema — name, url, description, author ✓
- `robots.txt` — allows all crawlers, points to sitemap ✓
- Sitemap (2 URLs: main `priority:1.0` + `/urdf-viewer/` `priority:0.9`) with `<lastmod>2026-06-27</lastmod>` — submitted to Google Search Console ✓
- Favicons — SVG, 192px, 48px, 32px, apple-touch-icon ✓
- Hero image alt text fixed (`ARM ON QUADRUPED.webp`) ✓
- Twitter handle: @amritj2002 (account exists but not active — tagged for attribution only) ✓

**Pending:** If a proper landscape `1200×630px` OG image is created, update `og:image`, dimensions, `twitter:image`, and switch `twitter:card` to `summary_large_image`. Current headshot (`og-preview.jpg`) is portrait 413×531 — works for `summary` card.

---

## Preload Hints (in `<head>`)
Three.js scripts, URDF file, and all 7 STL meshes are preloaded so the browser fetches them immediately on HTML parse.

---

## Performance Notes
- STL files from SolidWorks are oversized — total ~40MB across 7 meshes. Worst offenders: p2_manipulator (11.29MB), p1/p3/y2 (~7MB each)
- Decimating to 20–30% triangle count would be the biggest load time improvement; use Meshmixer (free) or meshoptimizer.org
- ⚠ Cloudflare Pages has a 25MB per-asset limit — p2_manipulator.STL (11.29MB) is fine, but decimation is still strongly recommended before migrating
- `images/og-preview.jpg` (413×531px portrait headshot) is used in both the About section identity card (210px sidebar, `object-fit:cover`) and as the OG/Twitter/JSON-LD image. `landing-page-photo.webp` is no longer referenced in the site.

---

## Security / Privacy
- Repo should be **private** — options: GitHub Pro ($4/mo) or migrate to Cloudflare Pages (free, supports private repos)
- Deploy obfuscated HTML via obfuscator.io — keep readable source local only
- Strip all comments from deployed file
- URDF/STL files cannot be truly protected on a static host — pre-rendered video is the only real solution

## Hosting Options Considered
- **GitHub Pages (current)** — free, requires public repo, served from US (higher latency for India)
- **Cloudflare Pages (recommended)** — free, supports private repos, Indian edge nodes (Mumbai/Chennai/Bengaluru), ~200-500ms faster for Indian visitors. Caveats: 25MB per-asset limit (check STL sizes), 30-60s build delay on deploy, requires moving DNS to Cloudflare nameservers (one-time, up to 24hr propagation)

---

## Theme System
- Default: **dark mode**
- Toggle: sun/moon icon button in nav (next to Blog button), id=`themeToggle`
- Persisted via `localStorage` key `theme` (`'light'` or `'dark'`)
- Light mode activated via `html[data-theme="light"]` — overrides all CSS variables
- Light tokens: `--bg:#f0f4f8`, `--bg2:#e6edf5`, `--bg3:#dce5ef`, `--line:#b8cad8`, `--steel:#3d6080`, `--accent:#007aaa`, `--accent2:#e05515`, `--text:#1a2a3c`, `--text-dim:#4a6a84`, `--white:#0a1520`, `--grid-col:rgba(0,122,170,0.07)`
- Hardcoded overrides in light mode: nav background, urdf-row, loader, btn-p text, blog overlay scrim

---

## About Section — Current Structure

Three horizontal bands (restructured 2026-06-30):

1. **Identity card** (`.about-card`) — `grid-template-columns:210px 1fr`. Left: `.about-img` shows `og-preview.jpg` (413×531 headshot) filling full card height via `object-fit:cover; object-position:center top`. Right: `.about-bio` — 3 paragraphs, `padding:2rem 2.25rem`, vertically centred. Card uses XP card design language (bg3, border, border-radius:6px, glass hover).
   - Responsive ≤900px: stacks vertically, photo capped at `max-height:240px`

2. **Credentials row** (`.about-creds`) — `grid-template-columns:1fr 1fr; gap:1rem; margin-bottom:2.5rem`. Contains CSWP cert (cyan left border) and B.Tech (orange left border). `.about-creds .acert` overrides `margin-top:0`.
   - Responsive ≤900px: single column

3. **Awards** (`.sec-num--label` glass chip + `.alist`) — full section width. 5 `.aitem` cards, each clickable (opens photo in lightbox).

### Bio text (3 paragraphs)
- Para 1: KinesthetIQ founding engineer, building actuators/grippers/teleoperation from scratch — custom planetary gearboxes, BLDC actuators, bimanual devices
- Para 2: lineage — Mars Rover Manipal → IISc RBCCPS (legged robot manipulation) → Chirathe/ARTPARK (high-TRL quadrupeds). Iteration cycle.
- Para 3: larger goal — LBM training data, standardised work cells, deployable general-purpose manipulation

---

## Visual Design System

### Colour spectrum (added 2026-06-30)
Applied across the site using nth-child selectors — no HTML classes needed:

**Experience card dots / hover borders / date labels:**
- nth-child(1) KinesthetIQ: `--accent` cyan
- nth-child(2) Chirathe: `--accent2` orange
- nth-child(3) IISc: `--accent3` violet
- nth-child(4) Curiouz: `--steel`
- nth-child(5) Mars Rover: `--accent2` orange

**Skill block titles (`.sblk-ttl`):**
- nth-child(1): cyan | nth-child(2): orange | nth-child(3): violet | nth-child(4): steel

**Section header lines (`.sec-line`):**
- `#experience`: default (cyan via var(--line)) | `#skills`: violet | `#about`: orange | `#contact`: default

**Contact link hovers:**
- nth-child(1) phone: cyan | nth-child(2) email: orange | nth-child(3) LinkedIn: violet | nth-child(4) GitHub: steel

### Frosted glass treatments
- **XP card hover**: `box-shadow: 0 14px 40px rgba(0,0,0,.4), inset 0 1px 0 <accent-rgba>` — tinted to match card's dot colour
- **Skill blocks**: `0 6px 24px rgba(0,0,0,.25), inset 0 1px 0 <accent-rgba>`
- **Award items**: `0 6px 22px rgba(0,0,0,.25), inset 0 1px 0 rgba(255,107,53,.08)`
- **Contact links**: full glass tile — `backdrop-filter:blur(14px) saturate(1.4)`, tinted bg, bright inset top-rim, lifts 3px
- **`.sec-num--label`**: glass chip — `rgba(255,255,255,.04)` bg, hairline border, `blur(8px)` backdrop
- **Loader `.loader-glass`**: `blur(18px) saturate(1.4)`, `inset 0 1px 0 rgba(255,255,255,.13)`, ambient cyan glow

### Animation system (2026-06-30)
- **Entrance curve**: `cubic-bezier(0.22,1,0.36,1)` — used on all reveals, hero fadeUp, nav logo swap. Snappy deceleration, no overshoot.
- **Hover transitions**: `ease-out` — cards, tabs, buttons, links, tags
- **Scroll reveal** (`.reveal`): `translateY(30px) scale(0.97)` → `none`, `.7s` duration. IntersectionObserver threshold `.12`, stagger `i×55ms`
- **Hero fadeUp**: `translateY(28px) scale(0.97)` keyframe, `.7s` per element, delays .2/.32/.46/.6/.72s
- **No `all` transitions** — all transitions are explicit property lists for performance
- **Timeline glow**: `::before` 2px line + `::after` blurred pseudo (`width:10px, filter:blur(5px)`) carrying the same gradient

---

## Outstanding / Next Steps
- [ ] Hero role text — 3-sentence version live; review once more after a few days
- [ ] Blog — 3 draft posts exist; remove `data-draft="true"` to publish
- [ ] SolidWorks animation — export as video, integrate in hero
- [ ] Create landscape `1200×630px` OG image — update og:image, dimensions, twitter:image, switch twitter:card to `summary_large_image`
- [ ] Decimate STL meshes for faster load (total ~40MB; target 20–30% triangle count reduction)
- [ ] Make repo private — use GitHub Pro ($4/mo) or migrate to Cloudflare Pages (free, better for India latency)
- [ ] Obfuscate HTML via obfuscator.io before deploying to production
- [ ] `images/landing-page-photo.webp` — no longer used, can be deleted from the repo to reduce size

## Completed
- [x] **Hero role text rewritten** — 3 sentences: field-ready hardware, first-principles build (actuators/grippers/teleoperation rigs), every design decision toward robots that work anywhere (2026-07-02)
- [x] **Mars Rover career stepper** — `.career-track` replaces single `.xp-role`; horizontal dots+lines mini-timeline; Trainee/Engineer/Mechanical Head with date ranges; active dot glows orange; line between steps 2–3 fades grey→orange; vertical stack on mobile ≤480px (2026-07-02)
- [x] **Mars Rover card updated** — date Nov 2020–Sep 2023 (full tenure); 4 specific bullets: all-terrain wheel + planetary gearbox, 6-DOF arm + soil module, design cycles, 200+ applicant recruitment (2026-07-02)
- [x] **Card meta text scaled up** — xp-date .62→.78rem, xp-role 1.05→1.20rem, xp-co .70→.78rem; proportional responsive overrides at 768px and 480px (2026-07-02)
- [x] **Experience card layout** — grid 2fr 3fr → 3fr 2fr (wider text, shorter media panel with no aspect-ratio distortion); align-items:start; column divider via ::before on .xp-body at left:60%; border-right removed from .xp-pts (2026-07-02)
- [x] **Mobile optimisation pass 2** — nav-links gap stepped (2.5rem→1.5rem@900px→1rem@768px); 640px block moved before 480px in cascade; xp-tab text-overflow:ellipsis; video play icon 2rem on mobile; landscape hero collapses to auto height; lightbox mobile adjustments (safe-area-inset, larger tap targets, swipe nav, frame sizing); scroll-to-top respects safe-area-inset-bottom; mobile-menu max-height (100dvh-64px) + overflow-y:auto; hamburger: outside-click close + body scroll lock + resize close (2026-07-02)
- [x] **Mobile hamburger menu** — `.hamburger` button in `.nav-actions` (outside `.nav-links`) + `.mobile-menu` slide-down panel; visible ≤640px; includes nav anchors + Blog/Field Notes button; ESC + link-click close; active section synced with scroll (2026-07-02)
- [x] **Scroll-to-top button** — `.scroll-top` fixed button (bottom-right), appears after 400px scroll, glass style with cyan border/glow (2026-07-02)
- [x] **Touch/swipe carousel** — `touchstart`/`touchend` handlers on `.xp-viewer`; 40px threshold; respects video tab (no auto-advance after swipe in video tab) (2026-07-02)
- [x] **IntroAnim cancel on slider touch** — `introAnim=null` added in slider `input` handler; prevents intro animation from overriding manual joint positions (2026-07-02)
- [x] **GitHub link in contact** — 4th `.clink` pointing to `https://github.com/amritanshu-j`; steel hover colour (nth-child(4) rule) (2026-07-02)
- [x] **Nav restructure** — theme toggle moved out of `.nav-links` into new `.nav-actions` wrapper; always visible including on mobile (2026-07-02)
- [x] **Active nav underline** — `inset 0 -2px 0 var(--accent)` box-shadow on `.nav-links a.active` (2026-07-02)
- [x] **Skill tag hover glow** — `box-shadow:0 0 10px rgba(143,180,204,.12)` added to `.stag:hover` (2026-07-02)
- [x] **DNS prefetch for drive.google.com** — added to `<head>`; all Drive thumbnail images now prefetch DNS (2026-07-02)
- [x] **Sitemap lastmod** — updated to `2026-07-02` for both URLs (2026-07-02)
- [x] **About section restructured** — identity card (photo sidebar + bio) + credentials row + full-width awards list. Bio rewritten with specific language: KinesthetIQ/LBMs/ARTPARK/IISc. (2026-06-30)
- [x] **Frosted glass system** — loader glass card, XP card hover highlights, skill block highlights, contact link glass tiles, award item highlights, sec-num--label chip (2026-06-30)
- [x] **Animation refinement** — `cubic-bezier(0.22,1,0.36,1)` entrances, `ease-out` hovers, explicit transitions, scroll reveal scale+translate, tighter stagger (2026-06-30)
- [x] **Three-accent colour system** — `--accent3:#8b7cf6` violet added (light: `#5b4fd8`); colour spectrum across cards, skills, sections, contact links (2026-06-30)
- [x] **Timeline glow** — 2px line + blurred `::after` glow pseudo (cyan→violet gradient) (2026-06-30)
- [x] **KinesthetIQ Videos tab** — 4 Drive video entries (facade pattern: thumbnail shown, iframe only loads in lightbox on click) (2026-06-28)
- [x] **Loader glass card** — `.loader-glass` frosted panel, subtitle removed, display time extended to 2200ms (2026-06-30)
- [x] **Progress bar navigation** — replaced dot indicators; 2px animated bar + slide counter (X / N) (2026-06-28)
- [x] **URDF viewer fixed** — both hero and standalone viewer now use self-contained URDF parser. Fixed `||` → `??` for 0-value joint limits, `.trim()` on joint names (2026-06-27)
- [x] **Favicon PNGs created** — `favicon-192.png`, `favicon-48.png`, `favicon-32.png` (2026-06-27)
- [x] SEO — meta tags, OG, Twitter/X Card, JSON-LD Person + WebSite schemas (2026-06-26)
- [x] robots.txt + sitemap (2 URLs, lastmod 2026-07-02) (2026-06-27)
- [x] Dark/light mode toggle with localStorage persistence (2026-06-26)
- [x] Mobile optimisation — responsive breakpoints 900px / 768px / 640px / 480px + landscape; URDF as background (2026-06-27)
- [x] Standalone URDF Viewer — `urdf-viewer/index.html`, drag-drop, full controls, joint sliders, animate mode (2026-06-27)
- [x] Resume button in hero CTAs — Drive PDF id: 1zgP4bzyl2fhCIfZvKDf5tKxqWeEsjUkZ (2026-06-27)
- [x] Mars Rover card — YouTube embed (`LtKkaTUwOCQ`, starts 38s) (2026-06-27)
