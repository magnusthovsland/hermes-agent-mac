# Google Drive secret hand-off pattern

Use this when the user deliberately stores an integration credential in a Google Drive document and asks Hermes to retrieve it for local use.

## Workflow

1. Resolve the intended Drive location first: folder/path, exact document name, and whether the target is a Google Doc or an uploaded file.
2. Use the existing Google OAuth token if available, but report only scope/availability — never print token contents.
3. Search Drive by exact-ish name first, then broaden to `name contains ... or fullText contains ...` if needed. Include shared drives flags when using the Drive API.
4. If the file is a Google Doc, export it as `text/plain`; for uploaded text/CSV/Office files, use download or the relevant document extraction path.
5. Extract likely credential values locally, but do not echo the full secret in chat or logs. Display only masked prefixes/suffixes such as `phx_...RjQe`.
6. Validate the credential with the narrowest safe API call for the target service.
7. If useful for future sessions, save a local credential file under `~/.hermes/credentials/<service>_<project>.json` or equivalent, `chmod 600`, and include host/project metadata plus the source Drive doc ID. Never commit this file.
8. Verify with a harmless read-only query/status call and report real non-sensitive output.

## PostHog-specific notes

- Personal API keys generally start with `phx_`; project/client capture keys often start with `phc_`.
- A project-scoped PostHog personal key may return `403` on organization/project-list endpoints with a message like “API keys with scoped projects are only supported on project-based endpoints.” This does not mean the key is unusable; identify the scoped project via key metadata or known project ID and use `/api/projects/:project_id/...` endpoints.
- `GET /api/personal_api_keys/@current/` can safely confirm the key label, masked value, scopes, and `scoped_teams` without revealing the secret.
- For analytics verification, prefer aggregate HogQL via `/api/projects/:project_id/query/`, e.g. counts by event for the last 24 hours, rather than exporting raw persons/events.
