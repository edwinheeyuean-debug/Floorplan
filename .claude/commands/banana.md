Generate a cleaned-up floor plan and furniture layout design from $ARGUMENTS using Nano Banana.

## Step 1 — Extract floor plan

Read the floor-plan image at the path provided in $ARGUMENTS.
Extract a structured JSON description following the rules in `prompts/read_floorplan.md` and conforming to `schemas/floorplan.schema.json`.
All dimensions are in MM.
Verify the extraction is complete before proceeding. Do not call Nano Banana if extraction failed or produced no rooms.

## Step 2 — Generate clean floor plan

Call the `nano_banana_generate_image` MCP tool with:
- `prompt`: built from the extracted JSON (room list, adjacency, door/window counts)
- `style`: `"architectural_line_drawing"`
- `background`: `"white"`
- `line_color`: `"#1a1a1a"`
- `width`: `1024`
- `height`: `1024`
- `preserve_topology`: `true`
- `reference_json`: the full extraction JSON as a string

Save the clean floor plan image to `outputs/`.

## Step 3 — Ask for design brief (if not already provided)

If the user has not provided a design brief alongside the image, ask:
- Budget (SGD)
- Interior design style (e.g. modern luxury)
- Number of occupants
- Any specific requirements or rooms to prioritize

If a brief was provided with the command, proceed immediately to Step 4.

## Step 4 — Generate furniture layout and 3D render

Follow the full workflow in `prompts/generate_furniture_layout_and_3d_render.md`:

1. Validate the extracted floor plan JSON.
2. Create `furniture_design_json` based on the user brief.
3. Create a furniture schedule with dimensions in MM.
4. Call Nano Banana to generate the 2D furniture layout plan.
5. Call Nano Banana to generate the 3D rendered design from the same `furniture_design_json`.
6. Run the consistency check.
7. Save all outputs to `outputs/<project_name>/`.

Never generate the 3D render before the 2D layout plan is complete.
Never use different furniture between the 2D layout and the 3D render.
Never block doors, windows, circulation paths, or household shelter entrances.

## Step 5 — Return final result

Respond in this format:

```
## Furniture Design Generated

### Design Concept
<short summary>

### Budget Direction
<short budget summary>

### Files
- Clean floor plan: `outputs/clean_floorplan.png`
- Furniture design JSON: `outputs/<project_name>/furniture_design.json`
- Furniture schedule: `outputs/<project_name>/furniture_schedule.md`
- 2D furniture layout plan: `outputs/<project_name>/furniture_layout_plan.png`
- 3D rendered design: `outputs/<project_name>/rendered_3d_design.png`
- Consistency report: `outputs/<project_name>/consistency_report.md`

### Consistency Result
<pass/fail with any issues>

### Important Notes
<uncertainties, assumptions, items needing confirmation>
```

If Nano Banana is not connected at any step, report `status: mcp_unavailable` for that step and continue with the remaining steps where possible.
