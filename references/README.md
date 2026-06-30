# Reference Repositories

These repos are **reference material only** — do not copy them blindly.
Use them to improve Claude's reasoning about floor plan conventions, HDB context, detection output structure, and dimension extraction logic.

---

## HDB / Singapore Context

### `terminalai/EmbodiedAI`
https://github.com/terminalai/EmbodiedAI

Purpose: Semantic segmentation of Singapore HDB apartment interiors and building facades.
Use for: Understanding HDB spatial conventions, room naming, typical room proportions, HDB-specific architectural vocabulary.
Limitation: Not a production floor plan parser for official HDB PDFs. Use as context only.

---

## Floor Plan Structure Recognition

### `zlzeng/DeepFloorplan`
https://github.com/zlzeng/DeepFloorplan

Purpose: Multi-task network identifying walls, doors, windows, room types from raster floor plan images.
Use for: Understanding structured floor plan recognition output — what elements to detect, how room types are classified, how wall/door/window detection is structured.
Limitation: Academic research model. Not directly callable as a service here.

### `TINY-KE/FloorPlanParser`
https://github.com/TINY-KE/FloorPlanParser

Purpose: Outputs vectorized JSON with start/end points for walls, windows, doors, bay windows from a floor plan image.
Use for: Reference for JSON output format — wall segments as start/end coordinate pairs, window/door positions. Aligns well with `schemas/floorplan.schema.json`.
Limitation: Depends on their proprietary parser service/account. Not self-hostable without their backend.

---

## Dimension Extraction

### `jasoncobra3/Floorplan-Dimractor`
https://github.com/jasoncobra3/Floorplan-Dimractor

Purpose: Python pipeline for extracting dimensions and codes from architectural PDF floor plans. Converts various dimension formats to standardized measurements with spatial coordinates.
Use for: Reference for how to detect and parse HDB dimension annotations (in MM). Useful for improving `prompts/read_floorplan.md` logic on visible dimensions extraction.
HDB note: HDB floor plan dimensions are always in MM. The Dimractor pipeline handles multiple formats — for HDB, only MM format is relevant.

---

## Object Detection Style Reference

### `sanatladkat/floor-plan-object-detection`
https://github.com/sanatladkat/floor-plan-object-detection

Purpose: YOLOv8 object detection trained on columns, walls, doors, windows in floor plan images.
Use for: Understanding a bounding-box detection approach to floor plan elements. Useful if Claude needs to reason about where detected elements are spatially (bounding boxes vs line segments).

### `mageaustralia/FloorPlanAnalyzer`
https://github.com/mageaustralia/FloorPlanAnalyzer

Purpose: Experimental multi-method tool combining CV, YOLOv8, OCR, CubiCasa5K, scale calibration, room area calculation, SVG export, and JSON export.
Use for: Understanding scale calibration logic (pixel-to-mm conversion), room area calculation from detected geometry, and honest limitations of automated floor plan parsing.
Note: Explicitly states its own limitations — good reminder for Claude not to over-trust automated detection results.

---

## How Claude Should Use These

```text
1. EmbodiedAI        → HDB spatial vocabulary and conventions
2. DeepFloorplan     → What elements to extract and how to structure them
3. FloorPlanParser   → Vectorized JSON output format reference
4. Floorplan-Dimractor → How to find and parse MM dimension annotations
5. YOLOv8 / FloorPlanAnalyzer → Bounding box detection approach and honest limitations
```

Claude must still follow `prompts/read_floorplan.md` and `schemas/floorplan.schema.json` as the authoritative output format.
These references do not override HDB-specific rules in `prompts/generate_furniture_layout_and_3d_render.md`.
