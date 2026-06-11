# Floor Plan Claude Agent

This project analyzes architectural floor-plan images and generates cleaned-up floor-plan images.

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
