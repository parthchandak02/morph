# morph — Creative Brief

## Concept
A living geometric organism driven by real-time sea surface temperature. A central polygon morphs its side count with SST — cold waters produce tight triangles, warm waters bloom into many-sided stars. Wave height drives a pulsing heartbeat. Annotations inline on the shape explain the data.

**Narrative Pattern:** Annotation — data values labeled inline on the geometric shape itself

**Tier 2 Patterns (pick 2):** Hero Number + Mini-Render Legend

## Data Source
- API endpoint: `https://marine-api.open-meteo.com/v1/marine?latitude={lat}&longitude={lon}&daily=sea_surface_temperature_max,wave_height_max&timezone=auto&forecast_days=1`
- For 5 coastal cities: San Francisco (37.77,-122.42), Reykjavik (64.15,-21.95), Sydney (-33.87,151.21), Mumbai (19.08,72.88), Cape Town (-33.92,18.42)
- Response fields: `daily.sea_surface_temperature_max[0]`, `daily.wave_height_max[0]`
- Fallback: if API fails, use stored defaults for each city (show "fallback" in stamp)

## Visual Style
- Preset: Dark — Void (monochrome, untouched)
- Palette: locked JS const
```js
const P = {
  bg: '#0a0a0a',         // near-black
  secondary: '#141414',  // subtle layer
  accent: '#e0e0e0',     // off-white primary stroke
  glow: 'rgba(224,224,224,0.15)', // white glow
  text: 'rgba(255,255,255,0.5)',
  warmth: 'rgba(200,200,200,0.3)', // warm tint for labels
};
```
- Coherence tokens: strokeWeight: {hair:0.5, fine:1, medium:2, heavy:4}, alpha: {bg:0.08, field:0.25, active:0.7, label:0.45}, shapeLanguage: geometric

## Encoding Contract
| Data Field | Visual Channel | Range | Inverse Function | "Bigger means..." | Legend? |
|---|---|---|---|---|---|
| sst (sea surface temp) | polygon side count | 3-12 sides | SST = map(sides, 3,12, 0,35) | warmer water | yes (hero number + mini-legend) |
| wave_height | pulse speed (interval) | 800-3000ms | waveH = map(speed, 800,3000, 0,8) | taller waves = faster pulse | yes (mini-legend) |
| city | shape position | 5 fixed positions | lookup table | N/A (categorical) | yes (inline labels) |

## Communication Requirements
**Tier 1 (mandatory):**
1. Persistent Data Stamp — bottom bar: "Open-Meteo Marine · SST X°C · updated Xs ago · live/fallback"
2. Data Process Honesty — fallback labeled "fallback", never "live"
3. One-to-One Sensory Mapping — more sides = warmer water (intuitive: more complexity = more energy)

**Tier 2 (pick 2):**
4. Hero Number — current SST of selected city at 48-72px in center of its polygon, with city name label below at 11px
5. Mini-Render Legend — small row at bottom showing: △ (cold, 3 sides) — ⬡ (warm, 12 sides) with temp labels

**Coherence:**
6. Generative-art axes: strokeWeight scale via `SW` const, alpha hierarchy via `A` const, shapeLanguage = geometric (all polygons, no curves)
7. Motion tokens: --motion-fast:160ms, --motion-base:240ms, --motion-slow:360ms, --ease-out-expo
8. AI Slop Score < 30 — no Inter/Roboto/Open Sans, layered bg, at least pulsing animation

## Canvas Code Requirements (MANDATORY)
1. Single HTML file, no external dependencies, no CDN libraries
2. Config object `C` at top with all tunable values (cities array, base sizes, min/max sides, pulse range)
3. DPR-aware canvas setup: `Math.min(window.devicePixelRatio || 1, 2)`, `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)`
4. Canvas CSS: `position:fixed;top:0;left:0;width:100%;height:100%;display:block`
5. Touch guard pattern — `let tapping=false; ontouchstart` gate
6. First frame never blank — pre-render 5 polygons with fallback SST values
7. Realistic fallback data if API fails (stored per-city defaults)
8. Short description text (`#desc` div, bottom center, muted opacity, system font)
9. Full-viewport, mobile-responsive
10. Works offline after first load (no CDN deps)
11. `.nojekyll` file at project root

## Interaction Affordances
- On first visit, polygons pulse in with a staggered fade-in (--motion-slow each, 0.2s stagger)
- Tap a polygon to select it: hero number updates to that city's SST, quiet zone shows city name + SST + wave height
- Tappable polygons have a subtle glow cue
- After 3s inactivity, selected state fades back to ambient art

## Implementation Notes
- Layout: 5 polygons arranged in a subtle arc or spread across the canvas. Each polygon centered on its own area.
- Each polygon independently pulses (rotation + scale breathing) at a speed determined by its city's wave height
- Polygon side count = map(SST, 0, 35, 3, 12). Clamp to integers.
- Hero number: the SST value at 48-72px rendered in the center of the selected polygon
- Mini-legend: bottom area showing a small triangle (cold) and a small star (warm) with temperature labels
- Data stamp: bottom left, 10px system-ui, opacity 0.4
- The 5 cities should be evenly spread. On tap, the selected polygon gently scales up (1.1x) while others dim slightly (alpha 0.15 -> active: 0.7, field: 0.25)
