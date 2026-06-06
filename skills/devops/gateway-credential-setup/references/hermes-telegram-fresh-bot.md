# Hermes Telegram fresh-bot credential pitfall

Session lesson: during Hermes messaging-gateway setup, a redacted/env-template check was interpreted too strongly and the assistant claimed a Telegram token was already configured. The user corrected this: they had not set up a Hermes Telegram token and suspected any token might have come from an OpenClaw setup. They wanted a new bot for Hermes, not reuse of a legacy bot.

Reusable lesson:

- Treat credential provenance as part of setup, not an afterthought.
- Check for active, uncommented env vars separately from commented templates.
- Redacted tool output can make `# TELEGRAM_BOT_TOKEN=` look similar to an active credential; verify line state before claiming `SET`.
- If a user says they want a Hermes-specific bot, give the fresh BotFather flow and do not reuse OpenClaw/legacy tokens.
- If you create temporary backups of credential files while inspecting or editing, remove them or disclose their paths immediately.

Recommended response shape after correction:

1. Acknowledge the mistaken assumption.
2. State what is actually active: e.g. “No active `TELEGRAM_BOT_TOKEN` is set in Hermes right now.”
3. Provide fresh setup steps:
   - `@BotFather` → `/newbot`
   - write new token to `~/.hermes/.env` as `TELEGRAM_BOT_TOKEN=...`
   - `hermes gateway run` for test
   - `hermes gateway install && hermes gateway start && hermes gateway status` for service mode
   - `/sethome` inside the Telegram chat if desired
4. Avoid displaying the token.
