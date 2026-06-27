---
name: communication-platform-integrations
description: "Umbrella workflow for communication and social platform integrations: terminal email, X/Twitter via xurl, and Yuanbao group/DM interaction."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [communications, email, social-media, x, twitter, yuanbao, himalaya]
---

# Communication Platform Integrations

Use this skill when the user asks the agent to operate communication or social platforms rather than just draft text: mailbox actions through Himalaya, X/Twitter actions through `xurl`, or Yuanbao group/DM interaction through the gateway/toolset.

The shared class is high-side-effect communication. Always identify the account/channel/audience, distinguish draft/read-only work from sends/posts/replies/DMs, protect credentials, and verify delivery or retrieval with the platform's own response/status when available.

## Triage: choose the platform path

- **Email mailbox operations** → Himalaya CLI for IMAP/SMTP: list/search/read messages, compose/send drafts, manage folders/flags.
- **X/Twitter** → official `xurl` CLI for posting, replies, quotes, search, timelines, media upload, DMs, and raw v2 API access.
- **Yuanbao (元宝)** → gateway-native group/DM interaction: normal reply text is delivered to the chat; `@nickname` is converted into a real mention; use available Yuanbao tools for group info and members.

## Operating rules

1. **Separate read-only from outbound actions.** Searching timelines, listing mail, and querying group members are low-risk; sending email, posting, deleting, DMs, and public replies need explicit user intent.
2. **Identify target and account.** Confirm mailbox/profile, X app/account, group/user, or thread before outbound actions when ambiguous.
3. **Never expose secrets.** Do not print OAuth tokens, API keys, SMTP passwords, or account cookies. Report credential state as set/not set.
4. **Use native structured output.** Prefer JSON from `xurl`, Himalaya message IDs/folder names, and Yuanbao tool results over screenshots or guesses.
5. **Verify outcomes.** After sends/posts, capture message ID/post ID/API response or, for gateway replies, make the final text exactly what should be sent.
6. **Respect platform semantics.** A gateway reply can itself be the sent Yuanbao message; do not incorrectly claim a separate send tool is required.

## Labeled playbooks

### Himalaya email CLI

Use Himalaya for terminal mailbox management via IMAP/SMTP/Notmuch/Sendmail backends. Configuration frequently needs provider-specific folder aliases, especially Gmail's `[Gmail]/Sent Mail`; preserved references cover configuration and MML message composition.

### X/Twitter via xurl

Use `xurl` for official X API access. It supports shortcut commands and raw v2 endpoints for posts, timeline/search, engagement, follows/blocks/mutes, media upload, DMs, and multi-account workflows. Treat public posts, deletes, and DMs as high-side-effect actions.

### Yuanbao group/DM interaction

For Yuanbao, the assistant's response text is the outbound message in gateway contexts. Include `@nickname` directly when the user asks to mention someone; the gateway resolves it. Use Yuanbao tools to query group info/members when needed.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `himalaya`, `xurl`, and `yuanbao`.
