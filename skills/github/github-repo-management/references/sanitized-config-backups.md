# Sanitized Configuration Backups to GitHub

Use this pattern when the user asks to back up the current tool/app configuration and status to a GitHub repository while excluding direct secrets, credentials, logs, and session/runtime state.

## Preferred repository shape

For a repository whose purpose is "current known-good local setup", prefer a root-current-state layout and let Git history provide the timeline:

```text
README.md
SECURITY.md
manifest.json
config/
skills/
memories/
cron/
status/
inventory/
```

Use timestamped directories like `backups/YYYYMMDDTHHMMSSZ/` only when the user explicitly wants many snapshots side-by-side in the working tree. Otherwise timestamp folders create clutter and make the latest state harder to inspect; each commit is already a timestamped snapshot.

Keep `snapshot_utc` in `manifest.json` for traceability even with the root-current-state layout.

## Scope

Include, after sanitization:
- Sanitized configuration files, including useful config backups where present.
- Installed skills and their supporting files (`references/`, `templates/`, `scripts/`) as text, unless too large/binary.
- Built-in memory files such as `memories/MEMORY.md` and `memories/USER.md`, after token/secret redaction.
- Cron definitions and small cron outputs when useful, after redaction.
- Status command outputs after redaction.
- Environment variable names only, never values.
- Non-sensitive inventory of selected local paths.
- A manifest describing snapshot time, source path, included sections, skipped files, and excluded sensitive paths.
- A `SECURITY.md` or README section documenting what is intentionally excluded.

Exclude:
- `.env` values.
- `auth.json`, OAuth credential pools, API keys, bot tokens, cookies, private keys, passwords, direct credentials.
- OAuth token/client-secret JSON files such as `google_token.json` and `google_client_secret.json`.
- SQLite state/session databases (`state.db*`, `kanban.db*`) unless the user explicitly requests a separate secure/private state backup.
- Raw gateway/session transcripts, request dumps, logs, cache directories, PID/lock files, sandboxes, and dependency trees (`node_modules`, virtualenvs).
- Credentialed remotes or URLs containing username:token.

## Workflow

1. Create or update a local checkout of the target repo. Keep the remote URL clean: `https://github.com/owner/repo.git`.
2. Choose layout:
   - Default: root-current-state layout (`config/`, `skills/`, `memories/`, `cron/`, `status/`, `inventory/`). Remove/replace the previous generated tree so root reflects the latest state.
   - Archive mode: timestamped `backups/YYYYMMDDTHHMMSSZ/` only if explicitly desired.
3. Sanitize structured config recursively:
   - Redact keys matching token/secret/password/credential/api_key/auth/bearer/cookie/session/private_key/client_secret/webhook/bot_token.
   - Also regex-redact common token strings (`ghp_`, `github_pat_`, `AIza`, `sk-`, Slack `xox...`, bearer tokens, credentialed GitHub URLs, private-key blocks, JWT-like tokens, AWS access keys, Stripe-like keys).
4. Copy desired text trees such as `skills/`, `memories/`, and `cron/` through the sanitizer. Skip locks, caches, huge files, binary files, dependency trees, and runtime state.
5. Record `.env` variable names only. Parse the env file as text; do not `source` it just to read one token because local `.env` files may contain quoted paths or shell-incompatible lines.
6. Run status commands and sanitize stdout/stderr before writing them.
7. Run a local secret scan on the complete generated repo tree before `git add`/commit. Treat regex examples in documentation as false positives only after inspecting the matched line.
8. Commit with a clear message such as `chore: flatten Hermes backup and include skills memories`.
9. Push without storing credentials in git config or the remote URL.
10. Verify with `git ls-remote origin refs/heads/main` or the relevant branch ref.

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
- Repository and branch.
- Commit SHA.
- Layout used (root-current-state or timestamped archive).
- File categories and counts included.
- File categories intentionally excluded.
- Secret-scan result, including any inspected false positives.
- Remote ref verification result.
