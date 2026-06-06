# hermes-agent-mac backup

This repository stores a sanitized operational snapshot of the local Hermes Agent setup on Magnus's macOS machine.

Security policy:
- Do not commit `.env`, `auth.json`, SQLite session databases, logs, raw gateway dumps, OAuth tokens, bot tokens, API keys, cookies, private keys, or credentials.
- Backup files are generated through a sanitizer that redacts secret-like keys and token-like strings.
- The snapshot is intended for recovery/context, not for restoring credentials.

Latest snapshot: `backups/20260606T001306Z/`
