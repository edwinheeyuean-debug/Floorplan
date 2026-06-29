# Prompt: Read Floor Plan

You are an architectural drawing analyst. You will be given an image of an architectural floor plan. Your job is to extract every detectable element and return a single JSON object that conforms to the schema at `schemas/floorplan.schema.json`.

## Instructions

### What to extract

For each category below, extract all visible instances. If a category has no detectable members, return an empty array — do not omit the key.

**Dimensions**
- The dimensions in all the floorplan sent are all in "MM" milimetres.

**Rooms**
- Assign a unique `id` (e.g. `"room_1"`, `"room_2"`).
- Record the `label` exactly as printed on the drawing. If no label is printed, set `label` to `null`.
- Infer `type` from the label or visual cues (e.g. `"bedroom"`, `"bathroom"`, `"kitchen"`, `"living_room"`, `"hallway"`, `"stairwell"`, `"unknown"`).
- Record `approximate_area_m2` only if a dimension or scale bar allows a reasonable estimate. Otherwise set to `null`.
- Record bounding box as `bbox: { x, y, width, height }` in normalized image coordinates (0.0–1.0).
- "R.C. Flat Roof Above, at 7th floor only" There is a concrete roof slab above that external space instead of open sky at 7th Storey Only, this feature exists only for units on the 7th floor.

**Walls**
- Record each wall segment as a line: `{ id, start: {x, y}, end: {x, y}, thickness_px }` in normalized image coordinates.
- Do not record implied walls; only walls visible as drawn lines.

**Doors**
- Record position, swing arc direction, and which room IDs the door connects: `{ id, position: {x, y}, connects: ["room_1", "room_2"], swing_direction, door_type }`.
- `door_type`: `"single"`, `"double"`, `"sliding"`, `"folding"`, or `"unknown"`.
- `swing_direction`: compass bearing of the arc (e.g. `"NE"`) or `null` if undetectable.

**Windows**
- Record position and the wall segment it belongs to: `{ id, position: {x, y}, wall_id, width_normalized }`.

**Stairs**
- Record bounding box, direction of ascent if shown, and step count if labeled: `{ id, bbox, ascent_direction, step_count }`.

**Adjacency**
- List pairs of rooms that share a wall or are directly connected by a door: `[ { room_a, room_b, connection_type } ]`.
- `connection_type`: `"shared_wall"`, `"door"`, or `"open_passage"`.

**Visible Dimensions**
- Extract every labeled measurement exactly as written: `{ id, label, value, unit, applies_to }`.
- `applies_to`: the element ID the dimension refers to, or `null` if unclear.
- **Do not invent or estimate measurements.** Only record what is explicitly printed.

**Uncertainties**
- For any detection where you are not highly confident, add an entry: `{ element_id, element_type, reason }`.
- Reasons include: low contrast, partial occlusion, ambiguous symbol, overlapping text, image quality, unfamiliar convention.

### Coordinate system

All coordinates are normalized to the image dimensions:
- `x = 0.0` is the left edge, `x = 1.0` is the right edge.
- `y = 0.0` is the top edge, `y = 1.0` is the bottom edge.

### Output format

Return **only** the JSON object. No prose before or after. No markdown code fences. The output must be valid JSON that passes the schema at `schemas/floorplan.schema.json`.

```json
{
  "schema_version": "1.0",
  "source_image": "<filename or null>",
  "rooms": [...],
  "walls": [...],
  "doors": [...],
  "windows": [...],
  "stairs": [...],
  "adjacency": [...],
  "visible_dimensions": [...],
  "uncertainties": [...]
}
```

### What NOT to do

- Do not invent room types that cannot be inferred from labels or clear visual context.
- Do not assign area values unless a scale bar or explicit dimension makes calculation possible.
- Do not silently drop uncertain elements — extract them and flag them in `uncertainties`.
- Do not hallucinate labels; if text is illegible, set `label` to `null` and add an uncertainty entry.
