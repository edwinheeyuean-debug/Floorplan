# Floor Plan Claude Agent

This project analyzes architectural floor-plan images and generates cleaned-up floor-plan images, furniture layout plans, and 3D rendered interior designs.

## Main workflow

1. Inspect the input floor-plan image.
2. Extract a structured JSON description.
3. Identify uncertainty instead of guessing.
4. Only after extraction, call the Nano Banana/Gemini image tool if connected.
5. Preserve the original layout unless the user explicitly requests changes.

## Extract these elements

- exterior walls (thick lines)
- interior walls (thinner lines)
- structural / RC walls (very thick black — cannot be hacked)
- rooms and room labels
- doors (with swing direction)
- windows (position and wall reference)
- stairs
- openings and open passages
- visible dimensions (in MM for HDB)
- room adjacency
- non-habitable zones (bomb shelter, RC flat roof, air-con ledge)
- uncertain or unclear areas

## HDB-specific recognition rules

These apply to all Singapore HDB floor plans:

- **All dimensions are in MM.** Never assume metres or cm.
- **Thick black walls = reinforced concrete.** Cannot be hacked. Most commonly the bomb shelter (Apt. Shelter).
- **Store Pantry / Apt. Shelter** = bomb shelter. Reinforced concrete all sides. HDB regulations allow **storage use only** — not bedroom, not study, not gym.
- **R.C. Flat Roof Above (At 7th Storey Only)** = concrete slab above external space. Not habitable. Not open sky. Only present on 7th floor units.
- **Air-Con Ledge** = external mechanical zone for compressors. Not habitable. No furniture.
- **PLUG DROP** = main electrical/entry point, typically marks the unit entry side.
- **Standard HDB ceiling height = 2700mm.** No double-height or voids unless explicitly shown.
- When extracting room types, use the label printed on the drawing — do not invent types.

## Reference material for floor plan reasoning

See `references/README.md` for curated repos. Key references:

| Reference | Use for |
|---|---|
| `terminalai/EmbodiedAI` | HDB/Singapore spatial vocabulary and conventions |
| `zlzeng/DeepFloorplan` | Structured element detection (walls, doors, windows, room types) |
| `TINY-KE/FloorPlanParser` | Vectorized JSON output format (wall start/end, door/window positions) |
| `jasoncobra3/Floorplan-Dimractor` | MM dimension extraction from architectural PDFs |
| `mageaustralia/FloorPlanAnalyzer` | Scale calibration, area calculation, honest parser limitations |

Do not copy these repos. Use them as reasoning context only.
The authoritative output format is always `schemas/floorplan.schema.json`.

## JSON rules

Output JSON matching `schemas/floorplan.schema.json`.

Never invent exact measurements.
Use approximate positions such as `top-left`, `center`, `bottom-right` when coordinates are unavailable.
Put unclear items in `uncertainties`.

## 2D furniture layout plan rules

**Do not use AI image generation for the floor plan base.** AI generators cannot reliably reproduce correct room positions. Instead:

1. Use Python (matplotlib) to draw the floor plan programmatically from extracted coordinates.
2. Place furniture blocks at correct positions using room boundary coordinates.
3. Add dimension lines, room labels, and a scale bar (1:100).
4. Save as PNG.

This guarantees the layout matches the original floor plan exactly.
See `outputs/pine_cl_16143_modernlux/` for an example of a programmatically drawn plan.

## Image generation rules (3D renders only)

Use Nano Banana / Gemini Imagen for **3D rendered room renders only** — not for 2D floor plan layouts.

When generating 3D renders:
- Generate one eye-level perspective render per habitable room (living, dining, kitchen, each bedroom)
- Do NOT generate a single dollhouse/cutaway overview
- Show only furniture from that room's furniture schedule
- Show bomb shelter door (steel door, painted to match kitchen) in kitchen render — never render shelter interior
- Do not add rooms, windows, stairs, or features not in the floor plan
- Keep HDB ceiling height at 2700mm

If Nano Banana MCP is not connected, call Gemini Imagen API directly:
```
POST https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=<GOOGLE_AI_API_KEY>
{"instances": [{"prompt": "..."}], "parameters": {"sampleCount": 1, "aspectRatio": "1:1"}}
```
API key is in `.claude/env` as `NANO_BANANA_API_KEY`.

## Furniture layout and 3D render rules

See `prompts/generate_furniture_layout_and_3d_render.md` for the full workflow.

Key rules:
- Always extract the floor plan JSON before generating furniture layout or 3D render.
- Create a `furniture_design_json` before generating any images.
- The 2D furniture layout plan and 3D renders must use the same `furniture_design_json`.
- Never add furniture in the 3D render that is not listed in the furniture schedule.
- Never block doors, windows, circulation paths, or household shelter access.
- Always produce a furniture schedule with dimensions in MM.
- Always produce a consistency report.
- Do not over-design beyond the user's stated budget.
- Do not expose or request API keys.
- Bomb shelter = storage only. Never propose any other use.
