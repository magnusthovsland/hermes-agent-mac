---
name: creative-visual-artifacts
description: "Umbrella workflow for visual creative deliverables: HTML/SVG mockups, diagrams, infographics, ASCII art, p5.js/pretext demos, and design-system-inspired artifacts."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [creative, design, diagrams, infographics, html, svg, ascii, p5js, prototypes]
---

# Creative Visual Artifacts

Use this skill when the user asks for a visual or browser-deliverable creative artifact: architecture diagrams, hand-drawn diagrams, infographics, ASCII art, HTML mockups, landing pages, interactive sketches, design-system riffs, or typography/text-layout demos.

## Default workflow

1. **Choose the artifact class before coding.**
   - Architecture/cloud/system diagram → SVG/HTML diagram with labeled components and arrows.
   - Hand-drawn/workshop diagram → Excalidraw JSON when the user wants editable sketch style.
   - Infographic → choose a layout (bento, timeline, matrix, iceberg, funnel, roadmap, etc.) and a visual style.
   - Mockup/prototype/landing page → single-file HTML/CSS/JS by default.
   - Generative/interactive art → p5.js sketch or Pretext text-layout demo.
   - Terminal/text aesthetic → ASCII art via figlet/cowsay/boxes/image-to-ASCII.
2. **Clarify only when the output medium materially changes.** If the request says "diagram", default to a portable HTML/SVG file; if it says "editable", use Excalidraw.
3. **Produce a real artifact.** Write the HTML/JSON/text file, open or render it when possible, and report the path. Do not stop at a prompt.
4. **Verify visually or structurally.** Use browser screenshot/vision for HTML; validate JSON for Excalidraw; run export/render commands for p5.js when applicable.
5. **Preserve source data.** Put reusable examples, style notes, and package-specific recipes under `references/`; starter files under `templates/`; runnable exporters under `scripts/`.

## Artifact classes

### Architecture and SVG diagrams

Use HTML with inline SVG for polished dark-themed infrastructure diagrams. Prefer semantic groups, consistent spacing, readable labels, and a legend. Start from `references/packages/architecture-diagram/templates/template.html` when you need a quick scaffold.

### Excalidraw-style diagrams

Use Excalidraw JSON when the user wants hand-drawn, editable diagrams. Keep element IDs stable, use the documented color palette, and run/upload helpers from the preserved package if needed: `references/packages/excalidraw/`.

### Infographics

For dense explanatory visuals, select both a **layout** and a **style** explicitly. The archived Baoyu package under `references/packages/baoyu-infographic/` contains layout/style banks and a structured content template.

### HTML mockups and design-system riffs

For landing pages, dashboards, decks, or comparison mockups, produce single-file HTML with strong visual hierarchy and responsive CSS. Use the preserved design-system templates from `references/packages/popular-web-designs/templates/` for brand/style inspiration, not direct cloning.

### p5.js and Pretext demos

Use p5.js for canvas/WebGL/generative animation and Pretext for DOM-free text layout, kinetic typography, and text-as-geometry experiments. Keep demos single-file unless export tooling is requested. Preserved packages live at `references/packages/p5js/` and `references/packages/pretext/`.

### ASCII art

Use pyfiglet/cowsay/boxes for text banners and image-to-ASCII for raster sources. Favor monospaced-safe output and include width assumptions.

## Pitfalls

- Do not return only a prompt for another model unless the user explicitly wants a prompt.
- Do not claim visual quality without rendering or screenshotting when browser tools are available.
- Keep package-specific support files with their package copy under `references/packages/<old-skill>/` so links and scripts remain recoverable.
- For web artifacts, avoid external CDN dependencies unless the user asked for them or the runtime requires them.

## Preserved source packages

The following previously separate skills were consolidated here as package snapshots under `references/packages/`: `architecture-diagram`, `ascii-art`, `baoyu-infographic`, `claude-design`, `design-md`, `excalidraw`, `p5js`, `popular-web-designs`, `pretext`, and `sketch`.
