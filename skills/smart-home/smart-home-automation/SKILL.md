---
name: smart-home-automation
description: "Class-level umbrella for smart-home control workflows: lights, rooms, scenes, Hue/OpenHue operations, scheduled automations, and safe verification of home-device commands."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [smart-home, home-automation, hue, lights, iot, scenes, rooms, cron]
---

# Smart Home Automation

Use this skill when the user asks to control or inspect smart-home devices: lights, rooms, zones, scenes, schedules, and local IoT automation. Prefer the narrowest available tool or CLI that controls the target device family, verify exact resource names before changing state, and report only actions that actually succeeded.

## Safety and Verification Rules

1. Discover available integrations before acting when the target is ambiguous (Hue/OpenHue, Home Assistant, vendor CLI, etc.).
2. List resources before controlling named devices unless exact names are already known from the same session.
3. Treat home-device commands as real-world side effects: confirm scope for broad changes like "everything off" if rooms/devices are not clear.
4. For schedules, prefer cron jobs only after the desired time, recurrence, target, and action are unambiguous.
5. Verify command success with the CLI/tool output or a follow-up state query when available.

## Philips Hue via OpenHue CLI

Use OpenHue when controlling Philips Hue lights, rooms, and scenes through a local Hue Bridge.

### Prerequisites

The bridge must be on the same local network as the machine running Hermes. First run requires physically pressing the Hue Bridge button to pair.

Install examples:

```bash
# Linux pre-built binary
curl -sL https://github.com/openhue/openhue-cli/releases/latest/download/openhue-linux-amd64 -o ~/.local/bin/openhue && chmod +x ~/.local/bin/openhue

# macOS
brew install openhue/cli/openhue-cli
```

Check availability before use:

```bash
which openhue
openhue get light
```

### Discover Resources

Resource names can be case-sensitive. Use discovery commands to get exact names:

```bash
openhue get light       # List all lights
openhue get room        # List all rooms
openhue get scene       # List all scenes
```

### Control Lights

```bash
# Turn on/off
openhue set light "Bedroom Lamp" --on
openhue set light "Bedroom Lamp" --off

# Brightness (0-100)
openhue set light "Bedroom Lamp" --on --brightness 50

# Color temperature: warm to cool, 153-500 mirek
openhue set light "Bedroom Lamp" --on --temperature 300

# Color, only on color-capable bulbs
openhue set light "Bedroom Lamp" --on --color red
openhue set light "Bedroom Lamp" --on --rgb "#FF5500"
```

### Control Rooms and Scenes

```bash
openhue set room "Bedroom" --off
openhue set room "Bedroom" --on --brightness 30

openhue set scene "Relax" --room "Bedroom"
openhue set scene "Concentrate" --room "Office"
```

### Common Presets

```bash
# Bedtime: dim and warm
openhue set room "Bedroom" --on --brightness 20 --temperature 450

# Work mode: bright and cool
openhue set room "Office" --on --brightness 100 --temperature 250

# Movie mode: dim
openhue set room "Living Room" --on --brightness 10
```

For "everything off", first identify rooms/zones, then issue explicit commands such as:

```bash
openhue set room "Bedroom" --off
openhue set room "Office" --off
openhue set room "Living Room" --off
```

## Scheduled Lighting

OpenHue works well with Hermes cron for scheduled routines such as dimming at bedtime or brightening at wake. A cron job prompt should be self-contained and include:

- exact target rooms/lights/scenes,
- exact recurrence and timezone assumptions,
- the CLI command(s) to run,
- verification/reporting expectations.

## Pitfalls

- Colors only work on color-capable bulbs; white-only models may ignore color/RGB flags.
- First pairing is physical and cannot be completed by the agent alone without a person pressing the bridge button.
- Local-network availability matters; commands may fail when Hermes is running on a remote backend or container not on the home LAN.
- Do not infer room names from natural language if discovery output uses different exact names.
