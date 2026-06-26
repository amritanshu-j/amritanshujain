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
  favicon-192.png
  favicon-48.png
  favicon-32.png
URDF/
  urdf_manipulator_v2/
    urdf/urdf_manipulator_v2.SLDASM.urdf
    meshes/
      base_link.STL
      p1_manipulator.STL
      p2_manipulator.STL
      p3_manipulator.STL
      r1_manipulator.STL
      y1_manipulator.STL
      y2_manipulator.STL
```

---

## Design Tokens
```css
--bg: #080b10       --bg2: #0d1117      --bg3: #111820
--line: #1e2d3d     --steel: #8fb4cc    --accent: #00d4ff
--accent2: #ff6b35  --text: #c9d8e8     --text-dim: #5a7a94
--white: #eef4f9    --grid-col: rgba(0,212,255,0.12)
```

---

## Site Sections (scroll order)
- **Nav** — fixed, liquid glass, logo animates to full name on scroll, Blog slide-in panel
- **Hero** — tagline, name (Bebas Neue), role, CTAs, KinesthetIQ logo, Three.js URDF viewer
- **Experience (01)** — `.xp-card` per role, Photos/CAD tabs, carousel, lightbox
- **Skills (02)** — 4 blocks
- **About (03)** — bio, CSWP cert, B.Tech MIT Manipal, awards grid
- **Contact** — phone, email, LinkedIn
- **Footer** — © 2025 · Bengaluru, Karnataka · India

---

## URDF Viewer — Current State

### Canvas / Layout (desktop)
- Container: `.bp-layer` — `position:absolute; top:calc(50% + 16rem); right:-2%; transform:translateY(-50%); width:68%; height:125%; z-index:0; overflow:visible;`
- `#urdfControls` — `position:fixed; bottom:6rem; left:70%; z-index:2;` — scrollable with `max-height:calc(100vh - 6rem)`

### Canvas / Layout (mobile ≤768px)
- `.bp-layer` fills hero: `position:absolute; inset:0; width:100%; height:100%; opacity:0.45;`
- `#urdfControls` hidden (`display:none!important`)
- Camera auto-scaled to fit arm in frame based on canvas aspect ratio — adapts to any screen size
- Camera X offset `dist*0.26` shifts arm left on mobile
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
// robotMd is hoisted from buildRobot() and available in resize()
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
MeshStandardMaterial({ color: 0xb2bcc8, metalness: 0.72, roughness: 0.28 })
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

---

## SEO — Current State
All of the following are implemented (fully live on amritanshujain.com as of 2026-06-26):
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
- Sitemap with `<lastmod>` (2026-06-26) — submitted to Google Search Console ✓
- Favicons — SVG, 192px, 48px, 32px, apple-touch-icon ✓
- Hero image alt text fixed (`ARM ON QUADRUPED.webp`) ✓
- Twitter handle: @amritj2002 (account exists but not active — tagged for attribution only) ✓

**Pending:** If a proper landscape `1200×630px` OG image is created, update `og:image`, dimensions, `twitter:image`, and switch `twitter:card` to `summary_large_image`. Current headshot (`og-preview.jpg`) is portrait 413×531 — works for `summary` card.

---

## Preload Hints (in `<head>`)
Three.js scripts, URDF file, and all 7 STL meshes are preloaded so the browser fetches them immediately on HTML parse.

---

## Performance Notes
- STL files from SolidWorks are likely oversized — decimating meshes to 20–30% triangle count would be the biggest load time improvement
- Use Meshmixer (free) or meshoptimizer.org to reduce file sizes

---

## Security / Privacy
- Repo should be **private** (GitHub Pro or Cloudflare/Netlify free tier)
- Deploy obfuscated HTML via obfuscator.io — keep readable source local only
- Strip all comments from deployed file
- URDF/STL files cannot be truly protected on a static host — pre-rendered video is the only real solution

---

## Theme System
- Default: **dark mode**
- Toggle: sun/moon icon button in nav (next to Blog button), id=`themeToggle`
- Persisted via `localStorage` key `theme` (`'light'` or `'dark'`)
- Light mode activated via `html[data-theme="light"]` — overrides all CSS variables
- Light tokens: `--bg:#f0f4f8`, `--bg2:#e6edf5`, `--bg3:#dce5ef`, `--line:#b8cad8`, `--steel:#3d6080`, `--accent:#007aaa`, `--accent2:#e05515`, `--text:#1a2a3c`, `--text-dim:#4a6a84`, `--white:#0a1520`, `--grid-col:rgba(0,122,170,0.07)`
- Hardcoded overrides in light mode: nav background, urdf-row, loader, btn-p text, blog overlay scrim

---

## Outstanding / Next Steps
- [ ] KinesthetIQ experience card — add photos/CAD media
- [ ] Mars Rover Manipal experience card — add photos
- [ ] Blog — 3 draft posts exist; remove `data-draft="true"` to publish
- [ ] SolidWorks animation — export as video, integrate in hero
- [ ] Create landscape `1200×630px` OG image — update og:image, dimensions, twitter:image, switch twitter:card to `summary_large_image`
- [ ] Decimate STL meshes for faster load
- [ ] Make repo private
- [ ] Test joint slider limits against URDF file (set `lower`/`upper` attributes in URDF to match JS limitMap)

## Completed
- [x] SEO — all meta tags, OG, Twitter/X Card, JSON-LD Person + WebSite schemas fully implemented (2026-06-26)
- [x] robots.txt created (2026-06-26)
- [x] Sitemap lastmod added (2026-06-26)
- [x] Geo meta tags added — IN-KA / Bengaluru (2026-06-26)
- [x] Hero image alt text fixed (2026-06-26)
- [x] Dark/light mode toggle added with localStorage persistence (2026-06-26)
- [x] Footer copyright updated to 2026 (2026-06-26)
- [x] OG/Twitter/JSON-LD image updated to headshot `og-preview.jpg` 413×531px (2026-06-27)
- [x] Mobile optimisation — responsive breakpoints at 768px and 480px (2026-06-27)
- [x] Mobile URDF — shown as transparent background animation, sliders hidden, camera auto-scaled to screen size, arm shifted left (2026-06-27)
- [x] KinesthetIQ logo increased to 110px height (2026-06-27)
- [x] "Now working with" label font size increased (2026-06-27)
- [x] urdfControls switched to position:fixed, shifted upward to bottom:6rem (2026-06-27)
