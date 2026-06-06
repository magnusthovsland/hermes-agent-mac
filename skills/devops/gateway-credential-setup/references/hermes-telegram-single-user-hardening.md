# Hermes Telegram single-user hardening checklist

Use this reference when the user wants Hermes available only through their Telegram user and not exposed from the host machine.

## Verification pattern

1. Confirm active gateway and configured platforms without exposing tokens:
   - `hermes gateway status`
   - `hermes status --all`
   - report platform status and home channel, not secrets.
2. Confirm Telegram allowlist uses numeric user IDs:
   - `.env` should include `TELEGRAM_ALLOWED_USERS=<numeric_user_id>`.
   - If the setup is single-user, also set `GATEWAY_ALLOWED_USERS=<same_numeric_user_id>` as a belt-and-suspenders global allowlist.
3. Explicitly block allow-all modes:
   - `TELEGRAM_ALLOW_ALL_USERS=false`
   - `GATEWAY_ALLOW_ALL_USERS=false`
   - Check no `*_ALLOW_ALL_USERS=true|1|yes` variables exist for configured platforms.
4. Check gateway pairing state:
   - `hermes pairing list`
   - If unknown approved pairings exist, revoke them before declaring the gateway single-user.
5. Check local network exposure on the host:
   - inspect listening ports and process names (`lsof -nP -iTCP -sTCP:LISTEN` on macOS/Linux).
   - Hermes dashboard/API endpoints should bind to `127.0.0.1` unless the user explicitly wants remote access.
   - Treat `0.0.0.0`, `*:<port>`, Cloudflare Tunnel, ngrok, Tailscale Funnel, localtunnel, FRP, or reverse proxies as separate exposure to verify, even if not Hermes.
6. Confirm file permissions:
   - `~/.hermes` should be user-private.
   - `.env`, `config.yaml`, and `auth.json` should not be world-readable.

## Safe hardening values

For a single-user Telegram-only Hermes setup, prefer explicit values like:

```env
TELEGRAM_ALLOWED_USERS=<numeric_user_id>
GATEWAY_ALLOWED_USERS=<numeric_user_id>
TELEGRAM_ALLOW_ALL_USERS=false
GATEWAY_ALLOW_ALL_USERS=false
TELEGRAM_GROUP_ALLOWED_USERS=
TELEGRAM_GROUP_ALLOWED_CHATS=
TELEGRAM_ALLOWED_CHATS=
```

And in `config.yaml`:

```yaml
unauthorized_dm_behavior: ignore
telegram:
  allow_from:
    - '<numeric_user_id>'
  unauthorized_dm_behavior: ignore
  allowed_chats: []
  group_allowed_chats: []
  group_allow_from: []
privacy:
  redact_pii: true
security:
  redact_secrets: true
  allow_private_urls: false
```

## Restart caveat

Config/env edits generally require a gateway restart. If working from inside the gateway, `hermes gateway restart` may be refused to prevent restart loops. Tell the user to run it from a shell outside the gateway, or use a supported gateway slash command if available.

## Reporting guidance

Separate:

- Hermes exposure: Telegram-only, dashboard localhost, API/webhooks disabled/not configured.
- Host exposure: any unrelated Cloudflare/ngrok/Tailscale/reverse-proxy services on the machine.
- Active mitigation applied: exact non-secret config keys changed and backup paths.
- Remaining action: restart required, tunnel/service review required, or token rotation if suspicious.
