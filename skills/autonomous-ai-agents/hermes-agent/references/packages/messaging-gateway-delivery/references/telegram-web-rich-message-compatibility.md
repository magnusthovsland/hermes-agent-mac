# Telegram Web rich-message compatibility

Session-derived pattern from a Hermes update where Telegram Web displayed an unsupported-message placeholder for agent replies.

## Symptom

- User receives a Telegram Web card/placeholder saying the message is not supported.
- Gateway and Telegram Bot API may still report sends as successful.
- The agent may be able to respond, but the client cannot render the message format.

## Likely cause

Newer Telegram rich-message or streaming delivery paths can produce messages accepted by Bot API but not rendered by Telegram Web. This is a client-compatibility failure, not necessarily an LLM/provider failure.

## Recovery checklist

1. Inspect effective config, not only YAML:
   - `gateway.streaming.enabled`
   - `gateway.streaming.transport`
   - Telegram `PlatformConfig.extra.rich_messages`
2. For Telegram Web compatibility, force legacy/plain delivery:
   - `gateway.platforms.telegram.extra.rich_messages: false`
   - `gateway.streaming.enabled: false`
   - `gateway.streaming.transport: false`
   - optionally `gateway.platforms.telegram.extra.disable_link_previews: true`
3. Restart the gateway so config is reloaded.
4. Verify the gateway is actually restarted/connected using a fresh PID/timestamp and current `Connected to Telegram` log line.
5. Send a short plain test message to the affected chat.
6. Tell the user that previously sent unsupported rich messages may remain unreadable; the fix applies to new messages.

## Good verification signals

- Effective config shows rich/streaming flags disabled.
- Gateway log shows a fresh startup and `Connected to Telegram` after the change.
- A short direct test message sends successfully and appears as normal text.

## Pitfalls

- Repeated restarts during an active turn can create restart-interrupted auto-resume loops.
- Long streamed/edited replies during recovery can trigger Telegram flood control, hiding the original issue.
- `send_message`/Bot API success proves delivery to Telegram, not necessarily rendering in Telegram Web.
