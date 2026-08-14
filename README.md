# hermes-agent-mac

Sanitized backup of Magnus's local Hermes Agent setup on macOS.

This repo represents the latest known safe snapshot in the root tree. Historical snapshots are handled by Git commits, not timestamped directories.

Current snapshot UTC: `20260814T010040Z`

Included:
- sanitized Hermes config
- installed Hermes skills, sanitized as text
- Hermes memories/user profile, sanitized as text
- cron job definitions/status where present, sanitized as text
- operational status outputs
- selected non-sensitive inventory

Excluded:
- direct secrets, credentials, tokens, private keys and passwords
- raw `.env` values; only variable names are stored
- `auth.json`, OAuth token files, Google token/client-secret files
- state databases, sessions, logs, request dumps and runtime cache files
- generated dependency trees such as `node_modules` and virtualenvs

Restore note:
Use this as a reference/configuration backup. Credentials must be recreated separately through `hermes setup`, `hermes auth`, platform setup, or a secure password manager.
