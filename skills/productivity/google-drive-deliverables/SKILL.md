---
name: google-drive-deliverables
description: "Deliver reports, analyses, and Office/PDF artifacts to Magnus's Google Drive folder `Dokumenter Hermes`."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [google-drive, documents, reports, deliverables, magnus]
    related_skills: [google-workspace, powerpoint, nano-pdf]
    created_by: agent
---

# Google Drive Deliverables

Use this skill when Magnus asks for reports, analyses, Word/Excel/PowerPoint-style deliverables, PDFs, Markdown exports, CSVs, or other generated artifacts that should be saved to Google Drive.

## Default destination

- Google Drive folder: `Dokumenter Hermes`
- Folder ID: `1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`
- Folder URL: `https://drive.google.com/drive/folders/1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI`
- Local pointer: `~/.hermes/google_drive_hermes_folder.json`

## Workflow

1. Create the artifact locally first.
   - Prefer real files over pasted content for substantive deliverables.
   - Use appropriate formats: `.md`, `.pdf`, `.docx`, `.xlsx`, `.pptx`, `.csv`, `.html`.
2. Verify the local artifact exists and is non-empty.
3. Upload it to `Dokumenter Hermes` using the Google Workspace API script.
4. Verify upload metadata, including parent folder ID when possible.
5. Reply to Magnus with:
   - filename/title
   - Google Drive link
   - local path
   - short summary of content
   - any assumptions/limitations

## Commands

Use the Hermes venv Python when invoking the bundled Google Workspace scripts:

```bash
GAPI="/Users/mt-infinity/.hermes/hermes-agent/venv/bin/python3 ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/google_api.py"
FOLDER_ID="1zh27EtcFCWCkpLDX-XIJwovHP9y-dcLI"
$GAPI drive upload /path/to/artifact.pdf --parent "$FOLDER_ID"
```

Find/verify the destination folder:

```bash
$GAPI drive search "name='Dokumenter Hermes' and mimeType='application/vnd.google-apps.folder' and trashed=false" --raw-query --max 10
```

Create a test subfolder only when explicitly testing Drive write access:

```bash
$GAPI drive create-folder "Hermes test - $(date '+%Y-%m-%d %H.%M.%S')" --parent "$FOLDER_ID"
```

## Access model

Hermes reuses the existing OpenClaw Google Drive OAuth credentials instead of forcing Magnus through a new setup flow.

- Source token: `/Users/mt-infinity/clawd-shared/credentials/google_token.json`
- Source client secret: `/Users/mt-infinity/clawd-shared/credentials/google_ads_client_secret.json`
- Hermes token path: `/Users/mt-infinity/.hermes/google_token.json`
- Hermes client secret path: `/Users/mt-infinity/.hermes/google_client_secret.json`

Current reused token has Drive scopes, not native Docs/Sheets edit scopes. Practical consequence:

- OK: generate local `.docx`, `.xlsx`, `.pptx`, `.pdf`, `.md`, `.csv`, `.html` and upload to Drive.
- Not guaranteed without re-auth: create/edit native Google Docs/Sheets via Docs/Sheets APIs.

## Approval boundaries

Allowed without extra confirmation when Magnus has asked for a document deliverable:

- create a local artifact
- upload it to `Dokumenter Hermes`
- create a clearly named test subfolder if Magnus explicitly asks to test Drive access

Require explicit confirmation before:

- deleting Drive files/folders
- changing sharing permissions
- emailing files or links
- modifying existing Drive documents owned by humans
- uploading sensitive/PII-heavy content if the user did not clearly ask for Drive delivery

## Pitfalls

- Do not assume Google-native Docs/Sheets APIs are available just because Drive upload works. Check token scopes or use file upload formats.
- Do not use a generic `python3` if it resolves to an older system Python that cannot run the script syntax. Prefer the Hermes venv Python path above.
- Do not expose token or client-secret contents in chat or logs. Show only credential file paths and redacted/scope metadata.
- If Drive auth fails, consult `references/openclaw-google-drive-reuse.md` before starting a new OAuth setup.

## References

- `references/openclaw-google-drive-reuse.md` — how Hermes was connected to OpenClaw's existing Google Drive access and how to verify `Dokumenter Hermes`.
