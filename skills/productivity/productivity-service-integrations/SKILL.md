---
name: productivity-service-integrations
description: "Umbrella workflow for productivity service integrations: Airtable, Google Workspace, Notion, Obsidian, Maps/location intelligence, and Teams meeting pipeline operations."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [productivity, integrations, airtable, google-workspace, notion, obsidian, maps, teams]
---

# Productivity Service Integrations

Use this skill when the user asks the agent to operate productivity services or personal/work knowledge systems through APIs, CLIs, or filesystem tools: Airtable bases, Google Workspace, Notion workspaces, Obsidian vaults, map/location lookups, or the Teams meeting summary pipeline.

The class-level pattern is the same across these systems: identify the service and target workspace, verify credentials/access without exposing secrets, prefer the official CLI or narrow REST call, perform the requested read/write, and verify the resulting record/document/file/job with a second read or status command.

## Triage: choose the service path

- **Airtable records/bases** → REST API via `curl` using a PAT; check base access and field names before writes.
- **Google Workspace** → prefer the `gws` CLI when installed, otherwise use the preserved Python bridge for Gmail, Calendar, Drive, Docs, and Sheets. If the user intentionally placed an integration secret in Drive, follow `references/google-drive-secret-hand-off.md`: export/download, mask secrets, validate with a narrow read-only API call, and save local credentials with `chmod 600`.
- **PostHog / marketing analytics destinations** → use `references/posthog-analytics-google-ads.md` for read-only PostHog credential hand-off, HogQL diagnostics, Hog functions/destination inspection, and Google Ads Conversions mapping/log review.
- **Notion** → prefer the official `ntn` CLI on macOS/Linux; fall back to HTTP + `curl` everywhere. Always ensure the target page/database is shared with the integration.
- **Obsidian** → filesystem-first note operations against a resolved vault path; never pass unresolved `$OBSIDIAN_VAULT_PATH` to file tools.
- **Maps/location intelligence** → use free OpenStreetMap/Nominatim, Overpass, OSRM, and TimeAPI flows for geocoding, POIs, routes, and timezone questions.
- **Teams meeting pipeline** → operate via `hermes teams-pipeline` subcommands for meeting summaries, transcripts, recordings, action items, job replay, and Microsoft Graph subscriptions.

## Operating rules

1. **Resolve identity and scope first.** Know the base, workspace, vault, calendar, database, location, or meeting job before making writes.
2. **Check access explicitly.** A valid token can still lack base/page/file access; distinguish missing credential from missing share/scope.
3. **Protect secrets.** Report tokens as set/not set only; never echo PATs, OAuth tokens, Notion keys, or Google credentials.
4. **Prefer read-before-write.** Inspect schemas, field names, database properties, sheet tabs, vault paths, or pipeline job status before mutation.
5. **Verify after action.** Re-read the record/page/file/job or run the relevant status/list command and report real output identifiers.
6. **Keep package detail intact.** Preserved source packages live under `references/packages/<service>/`; use them when a task needs exact command syntax, setup steps, or provider quirks.

## Labeled playbooks

### Airtable

Use Airtable for base/table/record CRUD, filters, formula queries, upserts, attachment fields, and schema inspection. Important pitfall: PATs are scoped per-base; a good token on the wrong base returns `403`. Inspect field names and field types before create/update calls.

### Google Workspace

Use Google Workspace for Gmail search/send, Calendar events, Drive files, Docs, and Sheets. First-time setup is OAuth-based and may require the user to visit an auth URL. The preserved package includes `scripts/setup.py`, `scripts/google_api.py`, and Gmail search syntax reference.

When Google Drive is being used as a deliberate secure hand-off location for third-party credentials, use the Drive secret hand-off reference instead of ad-hoc shelling: `references/google-drive-secret-hand-off.md`. Key points: never echo full secrets, distinguish Google OAuth scope from target-service access, validate the retrieved credential against the target service, and persist local secrets only under `~/.hermes/credentials/` with restrictive file permissions.

### PostHog / marketing analytics destinations

Use PostHog for product/marketing analytics, HogQL event diagnostics, persons/cohorts/definitions, and Data pipeline → Destinations checks such as Google Ads Conversions. Follow `references/posthog-analytics-google-ads.md` for exact endpoint patterns and queries.

Critical rules:
- Keep PostHog personal API keys secret; report only masked key hints/scopes and save local credentials with `chmod 600`.
- For Google Ads Conversions, inspect Hog functions (`/api/environments/:id/hog_functions/`) rather than guessing from dashboards.
- Validate destination mappings against actual events with HogQL counts, including event filters, property filters, resolved `gclid`/`gbraid`/`wbraid`, conversion value, currency, and order ID.
- Backend events with `$process_person_profile = false` may not inherit `person.properties.gclid`; verify whether click IDs are present directly on the conversion event before concluding Google Ads is undercounting.

### Notion

Use Notion for pages, databases, markdown import/export, dashboards, file uploads, and Workers. Share the target page/database with the integration before assuming the token can see it. Use `ntn` when available for concise operations and HTTP fallback for portability.

### Obsidian

Use Obsidian for local note search/read/create/edit and wikilinks. Resolve the vault path from env or default before file operations, and treat vault paths with spaces as normal absolute paths requiring quoting only in shell commands.

### Maps and location

Use maps for geocoding, reverse geocoding, nearby POIs, travel distance/time, directions, and timezone lookups. Default to free/open APIs and cache or rate-limit politely when doing repeated requests.

### Teams meeting pipeline

Use the Teams pipeline for meeting summary operations and Microsoft Graph subscription maintenance. Critical pitfall: Graph subscriptions expire in 72 hours; status checks should surface expiry and renewal state before debugging missing meetings.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `airtable`, `google-workspace`, `maps`, `notion`, `obsidian`, and `teams-meeting-pipeline`.
