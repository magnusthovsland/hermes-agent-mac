---
name: messaging-gateway-delivery
description: Class-level playbook for verifying and troubleshooting AI-agent messaging gateway delivery across Telegram and other chat clients, especially after updates or platform formatting changes.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [gateway, messaging, telegram, delivery, compatibility, troubleshooting]
---

# Messaging Gateway Delivery

Use this when the user reports that agent replies are missing, unreadable, unsupported by the chat client, duplicated, stuck after restart, or otherwise not delivered correctly through a messaging gateway.

This skill supplements the protected `hermes-agent` skill: use that skill/docs for authoritative Hermes commands, then use this playbook for the delivery-verification and client-compatibility workflow.

## Workflow

1. **Establish the symptom precisely**
   - Distinguish server/API success from user-visible rendering.
   - Ask/inspect whether the problem is: no response, delayed response, unsupported-message placeholder, malformed Markdown, media failure, restart loop, or flood-control delay.
   - For Telegram Web specifically, an unsupported-message placeholder after an update often points to Bot API rich-message/streaming compatibility rather than model failure.

2. **Check runtime state and logs**
   - Confirm the gateway process is loaded/running.
   - Check recent gateway logs for:
     - platform connected/disconnected lines
     - send/edit failures
     - flood-control/retry-after errors
     - restart-interrupted auto-resume loops
     - streamed/final delivery suppression notes
   - Do not treat an old startup line as current state; compare timestamps/PIDs.

3. **Check effective platform config, not just YAML text**
   - Load the effective gateway config with the platform config loader when available.
   - Confirm that the relevant flags actually reached `PlatformConfig.extra` and gateway streaming config.
   - For Telegram compatibility issues, verify:
     - `gateway.platforms.telegram.extra.rich_messages: false`
     - `gateway.streaming.enabled: false` when streaming itself causes client issues
     - `gateway.streaming.transport: false` when transport-level streaming causes client issues

4. **Prefer client-compatible fallback over clever formatting**
   - If the user's active client cannot render rich/streamed messages, force legacy/plain text even if the Bot API accepts the rich request.
   - For Telegram Web, plain text/Markdown-compatible sends are safer than Bot API rich messages.
   - Keep the fix minimal: change formatting/delivery mode before changing provider/model/session state.

5. **Restart only when config/code changes require it**
   - Gateway config changes usually require a gateway restart.
   - If operating from inside the gateway chat, avoid commands that are blocked to prevent restart loops; use the safe external/service path where available.
   - After restart, confirm a new process or fresh startup timestamp and platform connected line.

6. **Verify with a real user-visible send**
   - Send a short, plain test message to the affected platform/channel.
   - API success is necessary but not sufficient; report that the user should now see a normal text message, and state whether old unsupported placeholders were already-sent messages that cannot be retroactively fixed.

## Telegram Web compatibility quick fix

When Telegram Web shows “this message is not supported” for Hermes replies after an update:

```yaml
gateway:
  platforms:
    telegram:
      extra:
        rich_messages: false
        disable_link_previews: true
  streaming:
    enabled: false
    transport: false
```

Then restart the gateway and send a short plain test message.

## Pitfalls

- **Do not confuse Bot API acceptance with client rendering.** Telegram may accept a rich-message call while Telegram Web still displays an unsupported placeholder.
- **Do not chase model/provider issues first** when the logs show delivery completed but the client cannot render the message.
- **Do not over-restart blindly.** Repeated restarts can trigger auto-resume loops and extra blank/incomplete inbound events.
- **Avoid long streaming/edit bursts during recovery.** They can hit Telegram flood control and obscure the original compatibility problem.
- **Old bad messages remain bad.** A config fix only affects new messages; previously sent unsupported rich messages may still show unsupported placeholders.

## References

- `references/telegram-web-rich-message-compatibility.md` — session-derived notes for the Telegram Web unsupported-message failure mode and recovery checklist.
