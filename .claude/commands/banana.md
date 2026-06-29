Generate a cleaned-up floor plan image from $ARGUMENTS using Nano Banana.

Steps:
1. Read the floor-plan image at the path provided in $ARGUMENTS.
2. Extract a structured JSON description following the rules in `prompts/read_floorplan.md` and conforming to `schemas/floorplan.schema.json`.
3. Verify the extraction is complete before proceeding. Do not call Nano Banana if extraction failed or produced no rooms.
4. Call the `nano_banana_generate_image` MCP tool with:
   - `prompt`: a description built from the extracted JSON (room list, adjacency, door/window counts)
   - `style`: `"architectural_line_drawing"`
   - `background`: `"white"`
   - `line_color`: `"#1a1a1a"`
   - `width`: `1024`
   - `height`: `1024`
   - `preserve_topology`: `true`
   - `reference_json`: the full extraction JSON as a string
5. Save the output image to `outputs/` and return the file path or URL.

If the Nano Banana MCP tool is not connected, report `status: mcp_unavailable` and show the extracted JSON instead.
Never invent measurements. Never add or remove rooms from the layout.
