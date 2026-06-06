# Hermes Telegram allowlist debugging

Session pattern: Telegram connected successfully, but the user received no response from the bot. Gateway logs showed `Unauthorized user: <numeric_id> (<name>) on telegram`.

Key lessons:

- `TELEGRAM_ALLOWED_USERS` should be the numeric Telegram user ID in normal Hermes auth flows, not the visible username.
- A visible username like `sungam81` can be useful for humans but may not match `source.user_id` in the gateway.
- The fastest way to discover the correct ID is to ask the user to send any message to the bot, then inspect `~/.hermes/logs/gateway.log` for:

  ```text
  Unauthorized user: 7575580066 (Example Name) on telegram
  ```

- Put the numeric value into `.env`:

  ```env
  TELEGRAM_ALLOWED_USERS=7575580066
  ```

- Restart gateway after editing `.env` and verify fresh logs show:

  ```text
  Connected to Telegram (polling mode)
  ✓ telegram connected
  Gateway running with 1 platform(s)
  ```

- If old background watch notifications mention unauthorized users, compare timestamps. They may be stale notifications from a previous gateway run.

Bot identity verification:

- To confirm the token belongs to the intended bot without exposing the token, load the token from `.env` internally and call Telegram `getMe`.
- Report only non-secret fields: `bot_username`, `bot_name`, `bot_id`.

`/sethome` explanation:

- `/sethome` stores the current Telegram chat as Hermes's default delivery destination.
- This supports cron output, background-task completion notifications, and other gateway-originated messages.
- It does not merge the Telegram chat with the current CLI session; Telegram gets its own session under the same Hermes profile.
