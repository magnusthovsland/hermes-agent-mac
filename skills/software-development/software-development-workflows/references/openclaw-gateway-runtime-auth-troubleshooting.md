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

## Auth migration and `doctor --fix` pitfall

During legacy `openai-codex/*` to `openai/*` migrations, the config/session routes may be repaired while the per-agent auth store no longer contains a usable OpenAI profile. `models status` can look partially healthy because a profile exists in one layer, while `Runtime auth` still reports missing.

`openclaw doctor --fix` may classify an agent-local OAuth profile as a stale shadow, remove it, and report that the agent will inherit main auth. Do not assume this inheritance works at runtime: a subsequent agent turn can still fail with `No API key found for provider "openai"` against that agent's `openclaw-agent.sqlite`.

Safer resolution:

- Before `doctor --fix`, back up `~/.openclaw/openclaw.json` and every affected agent's `openclaw-agent.sqlite`; record the backup directory in the final report.
- After the fix, verify profile presence without exposing credentials. Inspect `auth_profile_store.store_json` and report only profile IDs/providers/types, never token values.
- Run a real non-delivered agent healthcheck immediately after restart. Config output and gateway health are insufficient.
- If `doctor --fix` removed a previously working profile and runtime auth fails, stop the gateway, preserve the broken post-fix DB, restore the known-good pre-fix SQLite backup with mode `0600`, restart, and retest.
- Prefer `openclaw models auth login --provider openai --agent <id>` or `openclaw configure` when a human can complete auth.
- If recovering a known-good local migration and the user has asked for direct repair, inspect OpenClaw-created auth backups such as `auth-profiles.json.sqlite-import.*.bak` or `auth-profiles.json.openai-provider-unification.*.bak` and restore only the relevant provider profile into each affected agent auth store.
- After repair, `openclaw models status --agent <id>` should list an OpenAI OAuth/token profile and the agent healthcheck metadata must show the intended provider/model.

## Model-default precedence and runtime verification

`openclaw models set <provider/model>` updates the global default only. It does **not** override an explicit per-agent model in `agents.list[*].model`; `openclaw models status --agent <id> --json` reports `modelConfig.defaultSource: "agent"` when that override wins. Also note that `openclaw models --agent <id> set ...` is rejected—the models setter does not support per-agent writes.

When changing the effective model for the default agent:

1. Set the global default:
   - `openclaw models set openai/gpt-5.6-sol`
2. Inspect precedence:
   - `openclaw models status --agent main --json`
   - `openclaw config get agents.list`
3. If `main` has an explicit override, update the matching list entry (verify the index from `agents.list` first):
   - `openclaw config set 'agents.list[0].model' 'openai/gpt-5.6-sol' --dry-run`
   - `openclaw config set 'agents.list[0].model' 'openai/gpt-5.6-sol'`
4. Restart the gateway when runtime reload is uncertain, then run the tiny agent healthcheck.
5. Treat config output as intent only. Success requires the healthcheck metadata to report the expected `provider` and `model`; a text reply alone can still come from an old per-agent override.

Do not silently change sibling agents when the user asks only for the default agent. Report which agents remain on the old model.

## Stable-channel model-support checks

When a user requires both a new model and stable-only software, verify those constraints independently before updating:

1. Query authoritative npm dist-tags (`npm view openclaw dist-tags --json`) and active channel (`openclaw update status --json`). Do not assume the numerically newest installed build is stable.
2. Inspect the exact stable package, not the currently installed beta, for model identifiers/capability code. A safe pattern is `npm pack openclaw@<stable-version>` in a temporary directory followed by a content search for the model family.
3. If stable lacks support but the installed beta has it, do not downgrade or upgrade automatically: explain that stable-only and model support conflict and ask which outcome to prioritize.
4. If the user chooses model support, keep the existing compatible build and change only the model configuration. If they choose stable, warn that model support will be lost before switching channels.

### Catalog support is not runtime support

A model appearing in `openclaw models list`, provider docs, or plugin capability code proves only that OpenClaw recognizes the identifier. The bundled Codex app-server can still reject the request with `The '<model>' model requires a newer version of Codex`.

Before recommending an OpenClaw/plugin update as a model fix:

1. Run a real agent healthcheck and inspect returned provider/model metadata or the exact provider error.
2. Inspect the installed `@openclaw/codex` package's dependency on `@openai/codex`.
3. Inspect the candidate release package with `npm pack @openclaw/codex@<version>` and compare that pinned dependency.
4. If current and candidate releases pin the same Codex version, do not claim the candidate will fix runtime support merely because its OpenClaw version is newer.
5. Prefer restoring a known-working model while waiting for a release that demonstrably bundles a compatible runtime; do not take a beta solely on speculation when the user requested stable-only software.

## Reporting standard

Final report should distinguish:

- Gateway/process health.
- Channel transport health.
- Agent runtime/model/auth health.
- Exact repair actions taken and backup paths.
- Remaining warnings that are non-blocking vs warnings that still affect replies.
