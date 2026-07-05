# OpenClaw gateway runtime/auth troubleshooting

Use this reference when an OpenClaw instance is reachable but messages do not produce real agent replies, especially after a version update, `doctor --fix`, plugin refresh, or legacy `openai-codex/*` migration.

## Symptoms seen

- Gateway and dashboard are up (`/health` returns OK, port is listening), and Telegram/channel probes say `works`.
- The bot may acknowledge or send an error, but the agent does not complete.
- Logs show one of these classes of failure:
  - Runtime/model mismatch, e.g. `Requested agent harness "codex" does not support openai/gpt-5.5`.
  - Auth migration/regression, e.g. `No API key found for provider "openai"` even though the configured model uses the Codex harness.

## Fast triage sequence

1. Verify process and HTTP health:
   - `openclaw gateway status --deep`
   - `curl http://127.0.0.1:<port>/health`
2. Verify channel transport separately from agent execution:
   - `openclaw channels status --probe --deep`
   - Treat `running/connected/works` as proof that Telegram/polling is not the main blocker.
3. Inspect model/runtime/auth state:
   - `openclaw models status --agent <id>` for each affected agent.
   - Check the configured default model and any per-model `agentRuntime` overrides in `~/.openclaw/openclaw.json`.
4. Reproduce with a tiny non-delivered agent turn before declaring success:
   - `openclaw agent --agent <id> --session-key agent:<id>:healthcheck-$(date +%s) --message 'Reply only OK_HEALTHCHECK.' --timeout 240 --json`
   - Success should show `status: ok`, expected provider/model, and a final text payload.
5. Only after the healthcheck passes, re-check channel status and ask the user to test their normal chat path if needed.

## Repair pattern

Preferred repair order:

1. Backup config/state before mutations.
2. Run OpenClaw's own repair/update path first:
   - `openclaw doctor --fix`
   - `openclaw plugins update codex` or `openclaw plugins update --all`
   - `openclaw update --yes` when the user explicitly asked for update or version drift is likely relevant.
   - `openclaw update repair --yes` after a package update if plugin/state convergence warnings remain.
   - `openclaw gateway restart`
3. Re-run the healthcheck. Do not stop at gateway/channel success; a model-auth failure can still block replies.

## Auth migration pitfall

During legacy `openai-codex/*` to `openai/*` migrations, the config/session routes may be repaired while the per-agent auth store no longer contains a usable OpenAI profile. `models status` can look partially healthy because a profile exists in one layer, while `Runtime auth` still reports missing.

Safer resolution:

- Prefer `openclaw models auth login --provider openai --agent <id>` or `openclaw configure` when a human can complete auth.
- If recovering a known-good local migration and the user has asked for direct repair, inspect OpenClaw-created auth backups such as `auth-profiles.json.sqlite-import.*.bak` or `auth-profiles.json.openai-provider-unification.*.bak` and restore only the relevant provider profile into each affected agent auth store.
- Before manual SQLite repair, copy each `openclaw-agent.sqlite` to a timestamped backup.
- After repair, `openclaw models status --agent <id>` should list an OpenAI OAuth/token profile and the agent healthcheck must pass.

## Reporting standard

Final report should distinguish:

- Gateway/process health.
- Channel transport health.
- Agent runtime/model/auth health.
- Exact repair actions taken and backup paths.
- Remaining warnings that are non-blocking vs warnings that still affect replies.
