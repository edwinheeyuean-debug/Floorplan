# Floor Plan Claude Agent

This project analyzes architectural floor-plan images and generates cleaned-up floor-plan images, furniture layout plans, and 3D rendered interior designs.

## Main workflow

1. Inspect the input floor-plan image.
2. Extract a structured JSON description.
3. Identify uncertainty instead of guessing.
4. Only after extraction, call the Nano Banana/Gemini image tool if connected.
5. Preserve the original layout unless the user explicitly requests changes.

## Extract these elements

- exterior walls
- interior walls
- rooms
- room labels
- doors
- windows
- stairs
- openings
- visible dimensions
- room adjacency
- uncertain or unclear areas

## JSON rules

Output JSON matching `schemas/floorplan.schema.json`.

Never invent exact measurements.
Use approximate positions such as `top-left`, `center`, `bottom-right` when coordinates are unavailable.
Put unclear items in `uncertainties`.

## Image generation rules

When using Nano Banana:
- preserve the floor-plan layout
- keep walls straight and aligned
- keep labels readable
- do not add rooms unless asked
- do not change dimensions unless asked
- return the generated image path or URL

## Furniture layout and 3D render rules

See `prompts/generate_furniture_layout_and_3d_render.md` for the full workflow.

Key rules:
- Always extract the floor plan JSON before generating furniture layout or 3D render.
- Create a `furniture_design_json` before calling Nano Banana for any design image.
- The 2D furniture layout plan and 3D rendered design must use the same `furniture_design_json` — they must be consistent.
- Never add furniture in the 3D render that is not listed in the furniture schedule.
- Never block doors, windows, circulation paths, or household shelter access.
- Always produce a furniture schedule with dimensions in MM.
- Always produce a consistency report.
- Do not over-design beyond the user's stated budget.
- Do not expose or request API keys.
