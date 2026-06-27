# Infinity Drift / Ovio Notion handoff pattern

Use this reference when taking over or verifying Infinity Drift Notion access for Hermes/Edith after another agent has been operating it.

## Durable pattern

1. **Do not trust a token just because it exists.** First call `/v1/users/me` to identify which Notion integration the current environment is using.
2. **Verify access against the actual target database/page.** A valid token can still return 404 if the integration is not shared with the page/database.
3. **If a newer Hermes/Edith integration exists but lacks page sharing, prefer the already-shared operational integration unless Magnus asks for a new integration.** This avoids breaking working Notion operations while permissions are reconciled in the UI.
4. **Keep secrets out of docs and memory.** Store only credential paths and integration names/IDs; never paste token values.
5. **Create Hermes-owned operational files instead of depending on another agent workspace.** Copy or symlink relevant scripts/setup JSON into a Hermes-owned directory and patch scripts to read `~/.hermes/credentials/notion_token`.
6. **Run read-only verification before write/maintenance scripts.** Then run idempotent maintenance scripts and confirm that the output is empty/no-op where expected.

## Infinity Drift/Ovio known structure

- Hermes-owned handoff/workspace path: `~/.hermes/notion/infinity-drift/`
- Hermes token path: `~/.hermes/credentials/notion_token`
- Ovio maintenance script pattern: `scripts/notion_ensure_ovio_project_boards.py`
- For Ovio, project-specific task boards should live under ordinary page `Ovio – Oppgaveboards`, not inside `Ovio – Prosjekter`.
- `Ovio – Prosjekter` should contain only real projects; board helper pages inside the database become erroneous rows.

## Verification checklist

- `/v1/users/me` returns the intended integration.
- Querying `Ovio – Prosjekter` succeeds and returns only the real projects.
- Each project has an `Oppgaveboard` URL.
- Ovio top page has child page `Ovio – Oppgaveboards`.
- Running the idempotent board maintenance script creates no unexpected pages/views and trashes no unexpected rows.
