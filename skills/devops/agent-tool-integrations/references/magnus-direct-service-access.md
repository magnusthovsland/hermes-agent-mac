# Magnus / Infinity Drift direct service access

Use this reference when Magnus asks to connect or verify agent access to individual services instead of using a broad integration aggregator.

## Standing preference

Magnus rejected broad Composio-style aggregation for sensitive company systems because a broker compromise could expose too many services at once. Prefer direct service-specific credentials/skills with least privilege, explicit token names, and explicit approval before writes or external/destructive actions.

## Global approval boundary

Allowed when task scope is clear:

- Read metadata/content needed for analysis.
- Generate local or Google Drive deliverables.
- Create clearly internal AI/test artifacts only when Magnus asked for a deliverable or test.

Require explicit approval:

- Sending/posting messages or email.
- Sharing documents externally.
- Updating/deleting production data.
- Writing to Airtable/Notion beyond an agreed AI Inbox/test target.
- Pushing commits, opening/commenting PRs, changing GitHub issues, triggering workflows, merging, deploys.
- AWS, finance, payment, inkasso, or production actions.

## Credentials pattern

Store secrets in `~/.hermes/.env`; do not paste values into chat. Suggested variable names:

- `NOTION_API_KEY`
- `AIRTABLE_API_KEY`
- `GITHUB_TOKEN` for the default/personal GitHub token
- `GITHUB_TOKEN_INFINITY_DRIFT` for Infinity Drift/Wright repos

When multiple GitHub tokens exist, do not rely on global `gh auth login` switching. Use per-command environment selection so the chosen token is explicit for each operation.

## Verification probes

Never print tokens. Verify with harmless metadata calls only.

### Notion

- Skill: `productivity/notion`
- Expected env: `NOTION_API_KEY`
- Probe account metadata with `GET /v1/users/me`.
- Probe visible content with `POST /v1/search` and a small `page_size`.

If auth works but search returns 0 results, the integration probably has not been connected to any Notion page/database. Ask Magnus to open target pages/databases and choose `... -> Connect to -> <integration name>`.

Known verified non-secret details from setup:

- Notion bot/integration: `Edith - Infinity Drift`
- Workspace: `Infinity Drift`
- Workspace ID: `c42ab3a0-2d72-4f88-8d4d-d68619d342b3`

### Airtable

- Skill: `productivity/airtable`
- Expected env: `AIRTABLE_API_KEY`
- Probe bases with Airtable metadata endpoint `/v0/meta/bases`.
- Inspect schema before any data work with `/v0/meta/bases/{base_id}/tables`.

Known verified non-secret details:

- Accessible base: `Teoria - qa`
- Base ID: `appGUwlhwtViQEfRm`
- Permission level observed: `create` — treat as read-only unless Magnus explicitly approves writes.
- Tables observed:
  - `Klasse` (`tblFSuqm9ytiiO7nP`)
  - `Spørsmål og svar` (`tblNGjddFJIaPnr17`)
  - `Tema` (`tblL6eI5xfd3AQwDl`)
  - `Undertema` (`tbl17s0ll9r8VlVVw`)
  - `Tag` (`tbliCa1dCOkWboHMi`)
  - `Gammel Fagkode` (`tblyTs3hcIF6AJ2Tf`)

### GitHub

- Skill: `github-auth` plus repo/PR/issue skills as needed.
- Default token env: `GITHUB_TOKEN`
- Infinity/Wright token env: `GITHUB_TOKEN_INFINITY_DRIFT`
- Probe identity with `gh api user` using the chosen token.
- Probe visible repos with `gh api user/repos` or `gh repo list` using the chosen token.

Known verified non-secret details:

- Default token user: `magnusthovsland`
- Default visible repo: `magnusthovsland/hermes-agent-mac`
- Infinity token user observed: `magnusthovsland`
- Accessible private Wright/Infinity repos observed:
  - `WrightTrafikkskole/Wright-Web`
  - `WrightTrafikkskole/wright-batskole-frontend`
  - `WrightTrafikkskole/heimdal-trafikkskole-web`
  - `WrightTrafikkskole/infinity-drift-frontend`
  - `WrightTrafikkskole/wright-batskole-admin`
- Permissions observed on these repos included admin/maintain/push/triage/pull. Treat as operationally read-only unless Magnus approves write actions.
- Repos explicitly named `ovio` or `teoria` were not observed in the accessible repo list during verification; they may be named differently or require additional access.

### Google Drive deliverables

Use the Google Drive deliverable workflow for reports and document artifacts.

Known default folder:

- Folder name: `Dokumenter Hermes`
- Folder ID: `1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`
- Local pointer: `~/.hermes/google_drive_hermes_folder.json`

## Reporting pattern after verification

Summarize as:

- Service: configured/auth status
- Visible account/workspace/base/repo metadata
- Permissions observed
- What works now
- What is missing or needs user action
- Security notes and approval boundaries
- Suggested first read-only task

Do not include token values, OAuth secrets, API keys, or raw authorization headers.
