# SCB Territory Checker & MBYFL Boundary Tools

A set of static web tools for the **Salinas Colts & Broncos (SCB)** and the broader **Monterey Bay Youth Football League (MBYFL)**. No build system, no backend — three standalone HTML files hosted on Cloudflare Pages.

**Live site:** [check.coltsandbroncos.com](https://check.coltsandbroncos.com)

---

## Files

| File | Purpose | Audience |
|---|---|---|
| `index.html` | Territory checker — enter an address, find out if it's in SCB's service area | Public |
| `draw.html` | Single-boundary drawing tool — click points on a map, export coordinates | Admin |
| `unidraw.html` | Multi-boundary drawing tool — draw and label multiple team regions, bulk export | Admin |

---

## index.html — Territory Checker

Users type an address; the app geocodes it (California addresses only) and runs a point-in-polygon check against the SCB boundary. Shows a green confirmation with confetti if in-territory, or a red result with links to the correct MBYFL team if out.

**To update the boundary:**
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

---

## Deployment

Hosted on **Cloudflare Pages** under project `scb-checker`.

```bash
npx wrangler pages deploy . --project-name scb-checker
```

No `wrangler.toml` needed — pure static files, no Workers. Make sure `SCB-Logo.png` is present in the directory before deploying.

Check Wrangler login status:
```bash
wrangler whoami
```

---

## Stack

| Concern | Implementation |
|---|---|
| Maps | Leaflet.js 1.9.4, CartoDB Voyager tiles |
| Geocoding | Nominatim / OpenStreetMap (free, no API key) |
| Image export | leaflet-image v0.4.0 (`draw.html` only) |
| Hosting | Cloudflare Pages |

All dependencies load from CDN. No npm, no build step.