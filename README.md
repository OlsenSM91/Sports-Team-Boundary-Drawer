# MBYFL Territory Checker & Boundary Tools

A set of static web tools for the **Monterey Bay Youth Football League (MBYFL)** and the **Salinas Colts & Broncos (SCB)**. No build system, no backend — four standalone HTML files hosted on Cloudflare Pages.

**Live site:** [check.coltsandbroncos.com](https://check.coltsandbroncos.com)

---

## Files

| File | Purpose | Audience |
|---|---|---|
| `mbyfl.html` | Unified MBYFL team finder — enter an address, get matched to your team | Public |
| `index.html` | Legacy SCB-only territory checker | Public |
| `unidraw.html` | Multi-boundary drawing tool — draw and label all 11 team regions, bulk export | Admin |
| `draw.html` | Single-boundary drawing tool — click points on a map, export coordinates | Admin |

---

## mbyfl.html — MBYFL Territory Checker

Users type an address; the app geocodes it (California only) and runs a point-in-polygon check against all defined team boundaries. Results open in a modal popup:

- **Single match** — team card with the team's name, logo, and registration link; confetti in the team's colors
- **Overlapping territory** — side-by-side team picker so families can choose
- **No match** — "Outside MBYFL Boundaries" with a link to contact MBYFL

The map displays all team regions using SVG diagonal stripe fills. When two boundaries overlap, their different-angled stripes produce a natural cross-hatch pattern — no geometry clipping needed.

**All 11 MBYFL teams are defined.** Only the Salinas Colts & Broncos boundary is currently populated. To add the remaining team boundaries, use `unidraw.html` (see workflow below).

---

## unidraw.html — Multi-Boundary Tool

Built for drawing all MBYFL team regions. Boundaries are saved to browser `localStorage` and persist across page refreshes.

**Drawing a boundary:**
1. Enter a team name and pick a color in the toolbar
2. Click points on the map to trace the boundary (each point is a draggable numbered dot)
3. Drag any point to adjust the line in real time
4. Click **Save Boundary** — it appears in the sidebar list

**Managing saved boundaries:**
- 👁 toggle visibility on/off per boundary
- ✏ edit — reloads the boundary's points as draggable markers
- ✕ delete

**Import:** paste any mix of formats — `const name = [[lat,lng],...]`, a JSON object `{"Team": [...]}`, or `// Team Name` comments above arrays. Multiple boundaries are detected automatically. Click **Parse** to preview, then **Import All**.

**Export All:** outputs all saved boundaries as a JavaScript object or JSON. The JS format can be pasted back into the import box on a future session.

```js
const BOUNDARIES = {
  "Team Name": [[lat, lng], ...],
};
```

### Workflow: Adding a Team Boundary to mbyfl.html

1. Open `unidraw.html` in a browser
2. Draw the team boundary and label it with the exact team name
3. Click **Export All** → JS tab → copy the coordinate array for that team
4. In `mbyfl.html`, find the matching team entry in the `TEAMS` array
5. Replace `boundary: []` with the copied coordinate array
6. Deploy

---

## index.html — Legacy SCB Territory Checker

Users type an address; the app checks whether it falls within the SCB boundary. Shows a green confirmation with confetti if in-territory, or a red result with a link to MBYFL if out.

**To update the SCB boundary:**
1. Open `draw.html`, paste the current `BOUNDARY_COORDS` array from `index.html` into the import box
2. Adjust points on the map
3. Click **Export coords** and copy the output
4. Replace `BOUNDARY_COORDS` in `index.html` with the new array
5. Deploy

---

## draw.html — Single Boundary Tool

Click to place boundary points on the map. Each click adds a numbered dot; the polygon fills in as you go.

- **Undo** removes the last point
- **Paste coords** loads an existing coordinate array so you can visualize and adjust it
- **Generate map** locks the drawing and previews the final polygon
- **Export PNG** downloads a map image
- **View coords** copies the coordinate array to paste into `index.html`

---

## Deployment

Hosted on **Cloudflare Pages** under project `scb-checker`. Make sure `SCB-Logo.png` and `mbyfl_logo.png` are present in the directory before deploying — wrangler uploads the entire local directory.

```bash
npx wrangler pages deploy . --project-name scb-checker
```

Check Wrangler login status:
```bash
wrangler whoami
```

No `wrangler.toml` needed — pure static files, no Workers.

---

## Stack

| Concern | Implementation |
|---|---|
| Maps | Leaflet.js 1.9.4, CartoDB Dark tiles (mbyfl.html) / Voyager (index.html) |
| Geocoding | Nominatim / OpenStreetMap (free, no API key) |
| Image export | leaflet-image v0.4.0 (`draw.html` only) |
| Hosting | Cloudflare Pages |

All dependencies load from CDN. No npm, no build step.
