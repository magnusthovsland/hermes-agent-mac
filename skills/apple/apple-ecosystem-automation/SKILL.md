---
name: apple-ecosystem-automation
description: "Class-level umbrella for operating Apple/macOS personal apps and desktop automation: Notes, Reminders, Messages, Find My, and background computer-use workflows."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [apple, macos, notes, reminders, imessage, findmy, computer-use, automation]
    related_skills: [google-workspace, obsidian]
---

# Apple Ecosystem Automation

## Overview
Use this umbrella when a task involves Apple-native personal apps or macOS UI automation. Prefer exact CLI tools when available, and fall back to background `computer_use`/AppleScript/screenshot workflows only when Apple does not expose a stable CLI.

## When to Use
- The user asks to create/search/edit Apple Notes.
- The user asks to add/list/complete Apple Reminders or iCloud-synced personal todos.
- The user asks to send/read iMessage/SMS from macOS Messages.
- The user asks where an Apple device, AirTag, pet tag, bag, or item is in Find My.
- The user asks you to drive a macOS desktop app or website in the background without stealing focus.

## Tool Choice by Job

### Apple Notes (`memo`)
Prereqs: macOS Notes.app, iCloud signed in, `brew tap antoniorodr/memo && brew install antoniorodr/memo/memo`, Automation permission for Notes.

Typical commands:
```bash
memo notes                         # list notes
memo notes -f "Folder Name"        # folder filter
memo notes -s "query"              # search
memo notes -a "Title"              # quick create
memo notes -e                      # interactive edit
```
Use Apple Notes for cross-device user-facing notes. Use the memory tool for agent-only durable preferences/facts, and Obsidian for vault workflows.

### Apple Reminders (`remindctl`)
Prereqs: macOS Reminders.app, `brew install steipete/tap/remindctl`, permission granted via `remindctl authorize` if needed.

Typical commands:
```bash
remindctl status
remindctl today
remindctl week
remindctl all
remindctl add "Buy milk" --due tomorrow --list Personal
remindctl complete <id>
```
Clarify when “remind me” could mean an agent alert (cron job) rather than an iCloud Reminder. Calendar events belong in Calendar/Google Calendar, not Reminders.

### iMessage / SMS (`imsg`)
Prereqs: macOS Messages.app signed in, `brew install steipete/tap/imsg`, Full Disk Access + Automation permissions.

Typical commands:
```bash
imsg chats --limit 10 --json
imsg history --chat-id <id> --limit 20 --json
imsg send --to "+14155551212" --text "Hello"
imsg send --to "+14155551212" --text "See attached" --file /path/to/file
```
Sending messages is an external side effect. Confirm recipient/content if ambiguous or high impact. Do not bulk-message without explicit approval.

### Find My / AirTags
Apple exposes no official CLI. Use Find My.app with AppleScript, screenshots, and vision/computer-use.

Basic pattern:
```bash
osascript -e 'tell application "FindMy" to activate'
sleep 3
screencapture -w -o /tmp/findmy.png
```
Then analyze the screenshot. For richer UI control, prefer background computer-use if available. Report freshness/time shown in the app; Find My locations can be stale.

### Background macOS computer use
When the `computer_use` tool is available, capture first and click by element index rather than by coordinates:
```text
computer_use(action="capture", mode="som", app="Safari")
computer_use(action="click", element=7)
```
This is for UI workflows where a CLI/API is unavailable. It should not steal the user’s cursor, keyboard focus, or Space. Re-capture after every state-changing action.

## Safety and Verification
- Verify prerequisites before assuming a CLI exists.
- For write/send actions, verify the destination (folder/list/contact/device) and read back when possible.
- Distinguish user-facing Apple state from agent-only memory/session state.
- Avoid exposing phone numbers or message content unnecessarily in summaries.
- For UI automation, capture evidence before reporting success.
