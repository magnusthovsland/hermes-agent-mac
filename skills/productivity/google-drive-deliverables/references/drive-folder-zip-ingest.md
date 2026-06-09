# Drive folder zip ingest workflow

Use when Magnus provides a Google Drive folder link containing a large zip/export instead of uploading directly to Telegram or the default `Dokumenter Hermes` folder.

## Pattern

1. Extract the folder ID from the Drive URL, e.g. `https://drive.google.com/drive/.../folders/<FOLDER_ID>`.
2. Verify Drive auth/read access and folder metadata:
   ```bash
   GAPI="/Users/mt-infinity/.hermes/hermes-agent/venv/bin/python3 ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/google_api.py"
   $GAPI drive get "$FOLDER_ID"
   ```
3. List immediate children with a raw parent query:
   ```bash
   $GAPI drive search "'$FOLDER_ID' in parents and trashed=false" --raw-query --max 100
   ```
4. Pick the intended artifact by exact/near-exact filename and modified time. If there are duplicates such as `Kopi av ...`, prefer the newest only when it clearly matches the user’s named file.
5. Download to a task-specific local directory under `~/.hermes/...`, not to Downloads:
   ```bash
   OUTDIR="$HOME/.hermes/notion/exports/YYYY-MM-DD-short-name"
   mkdir -p "$OUTDIR"
   $GAPI drive download "$FILE_ID" --output "$OUTDIR/export.zip"
   ```
6. Verify it is really a zip before extracting:
   ```bash
   python3 - <<'PY'
   from pathlib import Path
   p = Path('.../export.zip')
   print(p.stat().st_size, p.read_bytes()[:4])  # zip starts with b'PK\x03\x04'
   PY
   ```
7. Handle nested zip exports: list/extract the outer zip first, then inspect for `*-Part-1.zip` or similar inner archives.

## Safer extraction for Notion exports

macOS `/usr/bin/unzip` can stumble on Notion export filenames containing Norwegian/Unicode characters and ask interactive questions or report misleading write errors. For Notion exports, prefer Python `zipfile` extraction with path sanitization/truncation over blind `unzip`.

Minimal pattern:

```python
import pathlib, shutil, zipfile
zip_path = pathlib.Path("export-or-part.zip")
out = pathlib.Path("unzipped")
if out.exists():
    shutil.rmtree(out)
out.mkdir(parents=True)

with zipfile.ZipFile(zip_path) as zf:
    for info in zf.infolist():
        parts = []
        for part in pathlib.PurePosixPath(info.filename).parts:
            if part in ("", ".", ".."): continue
            part = "".join(ch if ch not in ":\0" else "_" for ch in part)
            if len(part.encode("utf-8", "ignore")) > 180:
                p = pathlib.Path(part)
                part = p.stem[:120] + "__TRUNC__" + p.suffix[:20]
            parts.append(part)
        if not parts: continue
        dest = out.joinpath(*parts)
        if info.is_dir():
            dest.mkdir(parents=True, exist_ok=True)
            continue
        dest.parent.mkdir(parents=True, exist_ok=True)
        with zf.open(info) as src, open(dest, "wb") as dst:
            shutil.copyfileobj(src, dst)
```

## Inventory after extraction

For Notion/database exports, always report:

- local zip path and extracted path
- file/dir counts
- counts by extension
- CSV filenames, rows, headers, and key field distributions such as `Status`, `Type`, `Priority`, `Assignee`, `Tags`
- largest non-CSV/Markdown attachments

Keep migration advice separate from extraction facts. Do not import hundreds of exported tasks into the current Notion workspace without an explicit mapping/review step.
