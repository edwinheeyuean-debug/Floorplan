# Floor Plan Agent

This agent reads architectural floor plan images and optionally generates cleaned-up redrawn versions.

## Sequence

1. **Read first.** Always run the extraction step before generation. Never call Nano Banana without a completed JSON extraction from the same floor plan.
2. **Generate second.** Pass the extraction JSON to Nano Banana. If the tool is not available in the current session, stop after extraction and report `status: mcp_unavailable`.

## JSON Output Rules

- Return **only** valid JSON. No prose, no markdown fences, no explanation before or after.
- Every response must conform to `schemas/floorplan.schema.json`.
- All required keys must be present even if their value is an empty array.
- All coordinates are normalized to the image dimensions (0.0–1.0).
- Set `approximate_area_m2` to `null` unless a scale bar or explicit label makes calculation possible. Do not estimate.
- Record `visible_dimensions` exactly as printed. Do not invent or infer measurements.
- Any element detected with less than high confidence must appear in `uncertainties` with a reason. Never silently drop uncertain elements.

## Layout Preservation Rules

- Do not add, remove, or reposition any room, wall, door, window, or staircase.
- The generated image must match the topology of the extracted JSON exactly.
- Do not add furniture, fixtures, title blocks, stamps, or decoration not present in the source.
- Do not infer a north arrow unless one is visible in the source image.

## API Key Rules

- Never store API keys, tokens, or secrets in any file in this repository.
- Read keys from environment variables at runtime only: `GEMINI_API_KEY`, `NANO_BANANA_API_KEY`.
- If a required key is missing from the environment, abort and report the missing variable. Do not fall back to a hardcoded value.

## Prompts

| File | Purpose |
|---|---|
| `prompts/read_floorplan.md` | Full extraction instructions |
| `prompts/generate_clean_floorplan.md` | Generation instructions and Nano Banana call spec |

## Schema

See `schemas/floorplan.schema.json` for the complete extraction contract.
