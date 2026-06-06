# Reusing OpenClaw Google Drive access for Hermes

Session learning: Magnus preferred avoiding a new Google OAuth setup because OpenClaw already had access to the right Google account. Hermes was connected by reusing the existing local credential files.

## Credential files found

OpenClaw/shared credentials:

- `/Users/mt-infinity/clawd-shared/credentials/google_token.json`
- `/Users/mt-infinity/clawd-shared/credentials/google_ads_client_secret.json`

Hermes expected locations:

- `/Users/mt-infinity/.hermes/google_token.json`
- `/Users/mt-infinity/.hermes/google_client_secret.json`

Copy pattern:

```bash
install -m 600 /Users/mt-infinity/clawd-shared/credentials/google_token.json /Users/mt-infinity/.hermes/google_token.json
install -m 600 /Users/mt-infinity/clawd-shared/credentials/google_ads_client_secret.json /Users/mt-infinity/.hermes/google_client_secret.json
```

## Auth verification

Use the setup script from the Google Workspace skill:

```bash
GSETUP="/Users/mt-infinity/.hermes/hermes-agent/venv/bin/python3 ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/setup.py"
$GSETUP --check
```

Expected acceptable result:

```text
AUTHENTICATED: Token refreshed at /Users/mt-infinity/.hermes/google_token.json
```

A partial-auth warning is acceptable for Drive-only delivery if the missing scopes are Docs/Sheets/Gmail/Calendar. It means native Docs/Sheets editing is unavailable, not that Drive upload is broken.

## Drive folder verification

Use the Hermes venv Python for `google_api.py`:

```bash
GAPI="/Users/mt-infinity/.hermes/hermes-agent/venv/bin/python3 ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/google_api.py"
$GAPI drive search "name='Dokumenter Hermes' and mimeType='application/vnd.google-apps.folder' and trashed=false" --raw-query --max 10
```

Known result:

- Name: `Dokumenter Hermes`
- ID: `1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`
- URL: `https://drive.google.com/drive/folders/1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`

Local pointer file created:

```text
/Users/mt-infinity/.hermes/google_drive_hermes_folder.json
```

## Write verification performed

A test folder was created successfully under `Dokumenter Hermes`:

- Name: `Hermes test - 2026-06-06 01.52.37`
- ID: `1dB-mR4Yn9H6VM5ljG0gaBulIKHpMEevm`
- Parent: `1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`
- Owner shown by Drive metadata: `jarvis.pi314@gmail.com`

## Important operational notes

- The reused token currently has Drive scopes: `https://www.googleapis.com/auth/drive` and `https://www.googleapis.com/auth/drive.file`.
- It does not currently include native Google Docs/Sheets API scopes.
- Prefer creating local Office/PDF/Markdown artifacts and uploading them to Drive unless Magnus explicitly asks for native Docs/Sheets and approves re-authorization.
- Do not print raw credential JSON contents. Redact tokens/client secrets; showing paths, scopes, folder IDs, and file IDs is acceptable.
