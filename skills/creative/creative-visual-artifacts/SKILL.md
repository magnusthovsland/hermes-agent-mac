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

### ASCII art and ASCII video

Use pyfiglet/cowsay/boxes for text banners and image-to-ASCII for raster sources. Favor monospaced-safe output and include width assumptions. For animated ASCII output, use the preserved `ascii-video` package: it covers video/audio/image/generative inputs, colored ASCII MP4/GIF rendering, audio-reactive visualizers, scene composition, shader/effect recipes, and optimization/troubleshooting notes.

### ComfyUI generation workflows

Use ComfyUI when the user needs node-graph image/video/audio generation, inpainting, upscaling, workflow parameter injection, local/cloud execution, or model/node lifecycle management. Prefer `comfy-cli` for setup/lifecycle and direct REST/WebSocket calls for workflow runs. Keep workflow JSON in API format and verify server health before queueing jobs.

### Manim explainer videos

Use Manim CE when the deliverable is educational animation: math/concept explainers, algorithms, equation derivations, architecture animations, or 3Blue1Brown-style videos. Plan the narrative/aha moment before writing code, render actual video output, and use preserved references for scene planning, visual design, equations, graphs, 3D/camera, and troubleshooting.

### TouchDesigner real-time visuals

Use TouchDesigner via the twozero MCP when the user needs a live network, projection mapping, interactive installation, audio-reactive visuals, particles, panels, GLSL, or external data visuals. Do not guess operator parameter names; inspect op/par info and errors with MCP tools before setting parameters.

### Humanized creative text

Use the humanizer material when the visual artifact includes copy that must sound natural, non-corporate, or non-LLM-written. Strip AI-isms, vary rhythm, add concrete human voice, and calibrate from user-provided samples when available.

## Pitfalls

- Do not return only a prompt for another model unless the user explicitly wants a prompt.
- Do not claim visual quality without rendering or screenshotting when browser tools are available.
- Keep package-specific support files with their package copy under `references/packages/<old-skill>/` so links and scripts remain recoverable.
- For web artifacts, avoid external CDN dependencies unless the user asked for them or the runtime requires them.

## Preserved source packages

The following previously separate skills were consolidated here as package snapshots under `references/packages/`: `architecture-diagram`, `ascii-art`, `ascii-video`, `baoyu-infographic`, `claude-design`, `comfyui`, `design-md`, `excalidraw`, `humanizer`, `manim-video`, `p5js`, `popular-web-designs`, `pretext`, `sketch`, and `touchdesigner-mcp`.
