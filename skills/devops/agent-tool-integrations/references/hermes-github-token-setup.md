# Hermes GitHub token setup pattern

Use when a user asks where to set `GITHUB_TOKEN`/`GH_TOKEN` for Hermes, GitHub-backed backups, or GitHub tool access.

## Durable lesson

`hermes config set` is for structured Hermes settings in `config.yaml`, not for secrets. Do not run:

```bash
hermes config set GITHUB_TOKEN ...
```

Instead:

1. Locate the active Hermes env file:

```bash
hermes config env-path
```

2. Add or update the token in that env file, usually `~/.hermes/.env`:

```dotenv
GITHUB_TOKEN=[REDACTED]
# or, for gh CLI compatibility:
GH_TOKEN=[REDACTED]
```

3. Restart any long-running Hermes process that needs the new env, especially gateway/cron workers.

4. Verify with harmless read-only GitHub calls while avoiding token disclosure. Examples:

```bash
GH_TOKEN=[REDACTED]
GH_TOKEN=[REDACTED]
```

A persistent `gh auth login` session is not required if `GH_TOKEN`/`GITHUB_TOKEN` is supplied in the environment. Report account/repo/permission metadata only, never the token.

## Reporting pattern

Report:

- whether the token is set, without printing it
- which account/repo metadata read-only verification returned
- whether `gh` is permanently logged in separately, if relevant
- whether gateway/cron must be restarted to pick up the new env

Do not treat a successful API call as approval to push, comment, create issues, or write to GitHub. For Magnus, external writes still require explicit approval for that specific action.
