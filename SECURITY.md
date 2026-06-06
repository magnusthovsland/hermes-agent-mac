# Security policy for this backup repo

Never commit raw credentials or local runtime state.

Allowed:
- sanitized config values
- skills and memories after secret-pattern redaction
- status command output after redaction
- environment variable names only

Denied:
- `.env` values
- `auth.json`
- `google_token.json`, `google_client_secret.json`, and similar OAuth files
- `state.db`, `kanban.db`, session transcripts, logs, request dumps
- SSH/private keys, certificates, cookies, API keys, bot tokens, access tokens, passwords

Before every commit, run a secret scan over the complete repo tree.
