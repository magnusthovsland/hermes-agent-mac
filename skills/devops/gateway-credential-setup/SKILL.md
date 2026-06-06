---
name: gateway-credential-setup
description: Set up messaging gateway credentials for Hermes or similar agent gateways without reusing legacy tokens or confusing templates with active secrets.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [gateway, credentials, telegram, discord, slack, hermes, setup]
---

# Gateway Credential Setup

Use this when configuring or troubleshooting messaging-platform bots for an agent gateway, especially Telegram/Discord/Slack, and the user cares which bot/token is used.

## Core rule: prove credential provenance

Before saying a platform is configured, distinguish three different states:

1. A commented template line exists in `.env` or docs.
2. An active environment variable is set in the gateway runtime.
3. The active token belongs to the bot/platform instance the user wants.

Do not treat template lines, redacted search hits, or inherited credentials from another project as proof that the new gateway is configured.

## Workflow

1. Ask or infer whether the user wants a new platform identity or to reuse an existing one.
   - If they say new bot/new integration/fresh Hermes setup, create or request fresh credentials from the platform authority.
   - Do not migrate or reuse old OpenClaw/legacy bot tokens unless the user explicitly asks.
2. Check active configuration without exposing secrets.
   - Prefer reporting `SET`/`not set` only.
   - Avoid copying credential files into backups unless necessary; if a backup is created during cleanup, remove it or tell the user exactly where it is.
3. For Telegram, the fresh-bot path is:
   - Open Telegram → `@BotFather` → `/newbot`.
   - Choose display name and a unique username ending in `bot`.
   - Put the new token in the gateway env as `TELEGRAM_BOT_TOKEN=<new token>`.
   - Start with `hermes gateway run` for a foreground test.
   - After successful test: `hermes gateway install`, `hermes gateway start`, `hermes gateway status`.
   - In the Telegram chat, run `/sethome` if the bot should receive cron/home deliveries.
4. When troubleshooting, verify the gateway process separately from credential presence.
   - `hermes gateway status` tells whether the listener is running.
   - A token being present does not mean the gateway is running.
   - A running gateway does not prove it is using the desired new bot.
5. For Telegram authorization, use the numeric Telegram user ID, not the username.
   - `TELEGRAM_ALLOWED_USERS=sungam81` is usually wrong for Hermes auth.
   - Watch `~/.hermes/logs/gateway.log` after the user sends a message; `Unauthorized user: 757... (Name) on telegram` reveals the numeric ID to allowlist.
   - Alternatively, tell the user to ask `@userinfobot` for the numeric `Id:`.
6. Verify a Telegram bot token belongs to the intended bot without exposing the token.
   - Call Telegram `getMe` with the token from `.env` and report only `bot_username`, `bot_name`, and `bot_id`.
   - This catches “token exists but belongs to the wrong/legacy bot” and confirms fresh BotFather setup.
7. After editing `.env`, restart the gateway and verify with fresh logs.
   - Stop old gateway runs before starting a new one, then check for `✓ telegram connected` and absence of fresh `No messaging platforms enabled` / `No user allowlists configured` warnings.
   - Ignore stale background-process watch notifications from earlier runs unless the log timestamp is after the latest restart.

## Single-user Telegram hardening / security review

Use this when the user wants Hermes reachable only from their Telegram account and not exposed openly from the host machine.

1. Verify the active gateway and configured platforms without exposing secrets: `hermes gateway status`, `hermes status --all`, and sanitized `.env`/`config.yaml` inspection.
2. Require numeric Telegram user IDs, not usernames. For Magnus's current Hermes setup the known authorized Telegram user ID is `7575580066`, but still verify live config before changing anything.
3. Explicitly set both platform and global allowlists for defense in depth:
   - `TELEGRAM_ALLOWED_USERS=<numeric_user_id>`
   - `GATEWAY_ALLOWED_USERS=<numeric_user_id>`
   - `TELEGRAM_ALLOW_ALL_USERS=false`
   - `GATEWAY_ALLOW_ALL_USERS=false`
4. For single-user DM-only access, clear Telegram group/chat allowlists unless the user explicitly wants group use: `TELEGRAM_GROUP_ALLOWED_USERS=`, `TELEGRAM_GROUP_ALLOWED_CHATS=`, `TELEGRAM_ALLOWED_CHATS=`.
5. Prefer `unauthorized_dm_behavior: ignore` so unknown DMs do not receive pairing prompts or reveal that an agent gateway is available.
6. Check host exposure separately from Hermes exposure. Look for listening ports and tunnel/proxy processes such as Cloudflare Tunnel, ngrok, Tailscale Funnel, localtunnel, FRP, API server, or webhooks. Do not conflate an unrelated tunnel with Hermes, but report it as host exposure.
7. Check secret/config file permissions: `~/.hermes`, `.env`, `config.yaml`, and `auth.json` should be user-private.
8. After config/env edits, restart the gateway from outside the gateway process. If `hermes gateway restart` is blocked from inside Telegram/gateway, tell the user to run it in a shell.

See `references/hermes-telegram-single-user-hardening.md` for exact hardening values and reporting structure.

## Pitfalls

- Do not tell the user a Telegram token is configured based only on a commented `.env` template line.
- Do not conflate a Telegram bot username with a BotFather token: `TELEGRAM_BOT_TOKEN` must be the token, while bot usernames only identify the chat target.
- Do not conflate Telegram usernames with authorization IDs. Telegram allowlists generally need numeric user IDs; if username auth fails, inspect gateway logs for the `Unauthorized user: <id>` line.
- Do not conflate CLI chat with gateway chat. CLI Hermes does not automatically use Telegram/Slack/Discord credentials; the gateway process does.
- Do not silently reuse credentials from OpenClaw or another legacy agent. Treat legacy credentials as suspect until the user confirms reuse.
- Do not paste or reveal bot tokens in responses. Refer to them as set/not set or by bot username when available.

## `/sethome` in Hermes Telegram

- `/sethome` sets the current Telegram chat as the default home channel for Hermes deliveries.
- It is useful for cron/scheduled jobs, background-task completion messages, and gateway-originated notifications.
- It does not make the Telegram conversation the same session as the current CLI thread; Telegram has its own session history under the same Hermes profile.

## References

- `references/hermes-telegram-fresh-bot.md` — notes from a session where a Telegram template line was mistaken for an active Hermes token and the user wanted a new bot instead of OpenClaw credential reuse.
- `references/hermes-telegram-allowlist-debugging.md` — numeric Telegram user ID allowlist debugging, `getMe` bot verification, stale watch notifications, and `/sethome` semantics.
