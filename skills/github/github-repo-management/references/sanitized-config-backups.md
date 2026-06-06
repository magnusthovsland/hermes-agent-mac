# Sanitized Configuration Backups to GitHub

Use this pattern when the user asks to back up the current tool/app configuration and status to a GitHub repository while excluding secrets, credentials, logs, and session data.

## Scope

Include:
- Sanitized configuration files (e.g. `config.yaml` with secret-like keys redacted)
- Status command outputs after redaction
- Environment variable names only, never values
- Non-sensitive inventories of selected directories
- A manifest describing snapshot time, source path, included sections, and excluded sensitive paths
- A `SECURITY.md` or README section documenting what is intentionally excluded

Exclude:
- `.env` values
- `auth.json`, OAuth credential pools, API keys, bot tokens, cookies, private keys
- SQLite state/session databases
- raw gateway/session transcripts, request dumps, and logs
- credentialed remotes or URLs containing username:token

## Workflow

1. Create or update a local checkout of the target repo. Keep the remote URL clean: `https://github.com/owner/repo.git`.
2. Generate a timestamped snapshot directory such as `backups/YYYYMMDDTHHMMSSZ/`.
3. Sanitize structured config recursively:
   - Redact keys matching token/secret/password/credential/api_key/auth/bearer/cookie/session/private_key/client_secret/webhook/bot_token.
   - Also regex-redact common token strings (`ghp_`, `github_pat_`, `AIza`, `sk-`, Slack `xox...`, bearer tokens, credentialed GitHub URLs, private-key blocks).
4. Record `.env` variable names only. Parse the env file as text; do not `source` it just to read one token.
5. Run status commands and sanitize stdout/stderr before writing them.
6. Run a local secret scan on the snapshot before `git add`/commit. If any finding remains, stop and remove/redact the file before committing.
7. Commit with a clear message such as `chore: add sanitized Hermes macOS backup`.
8. Push without storing credentials in git config or the remote URL.
9. Verify with `git ls-remote origin refs/heads/main` or the relevant branch ref.

## Safe push with token but no persisted credential

If `gh auth login` is not persisted but a token is available from a secret store or `.env`, avoid embedding the token in the remote URL. For git over HTTPS, pass a one-shot authorization header:

```python
import base64, subprocess

auth = base64.b64encode(f"x-access-token:{token}".encode()).decode()
subprocess.run([
    "git",
    "-c", f"http.https://github.com/.extraheader=AUTHORIZATION: basic {auth}",
    "push", "-u", "origin", "main",
], cwd=repo_path, check=True)
```

Never print the token or the full command with the header value. Redact command output defensively.

## Verification checklist

Before final response, report:
- Repository and branch
- Commit SHA
- Snapshot path
- File categories included
- File categories intentionally excluded
- Secret-scan result
- Remote ref verification result
