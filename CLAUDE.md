# Floorplan Reader & Generator

An agent project for extracting structured data from architectural floor plan images and generating cleaned-up redrawn versions.

## Overview

This project does two things:

1. **Read** — Analyze a floor plan image and extract structured JSON (rooms, walls, doors, windows, stairs, adjacency, dimensions, uncertainties).
2. **Generate** — Use a Nano Banana image-generation MCP tool to produce a clean, redrawn version of the detected layout.

## Setup

### API Keys

**Never store API keys in the repository.** Set them as environment variables:

```bash
export GEMINI_API_KEY="your-key-here"
```

The agent reads this at runtime. If the key is missing, it will refuse to call the API rather than fail silently.

### MCP Tool: Nano Banana

The generation step requires a Nano Banana image-generation MCP tool registered in Claude Code's MCP config (`~/.claude/mcp.json` or project `.mcp.json`). When the tool is unavailable, the agent completes only the extraction step and notes the gap.

Example MCP config entry (fill in your server details):

```json
{
  "mcpServers": {
    "nano-banana": {
      "command": "npx",
      "args": ["-y", "nano-banana-mcp"],
      "env": {
        "NANO_BANANA_API_KEY": "${NANO_BANANA_API_KEY}"
      }
    }
  }
}
```

## Directory Layout

```
prompts/
  read_floorplan.md           # Prompt for extraction step
  generate_clean_floorplan.md # Prompt for generation step
schemas/
  floorplan.schema.json       # JSON Schema for extracted data
examples/                     # Sample floor plan images (checked in, no PII)
outputs/                      # Extraction JSON + generated images (git-ignored)
```

## Running

### Extract structured data from an image

```bash
claude -p prompts/read_floorplan.md --image path/to/floorplan.png
```

Output is written to `outputs/<filename>.json`.

### Generate a cleaned-up floor plan

```bash
claude -p prompts/generate_clean_floorplan.md --image path/to/floorplan.png
```

Output image is written to `outputs/<filename>_clean.png`.

## Design Rules

- **Do not invent measurements.** Only include dimensions visibly labeled in the source image.
- **Always mark uncertainty.** Any detection below high confidence must appear in the `uncertainties` array with a reason.
- **Preserve layout fidelity.** The generated image must reflect the extracted topology — do not add, remove, or relocate rooms.

## Schema

See `schemas/floorplan.schema.json` for the full extraction contract.
