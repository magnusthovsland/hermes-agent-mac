# Teoria QA English backfill reference

This reference captures a successful Airtable localization workflow for Teoria QA. Treat IDs and field names as examples to verify live before reuse.

## Airtable objects used

- Base: `appGUwlhwtViQEfRm` (`Teoria - qa`)
- Question table: `tblNGjddFJIaPnr17` (`Spørsmål og svar` / `Alle spørsmål og svar`)
- Production-ready filter used in QA: `{Klar til prod}=TRUE()`

Readable context tables from that job:

- Class: `tblFSuqm9ytiiO7nP`
- Theme: `tblL6eI5xfd3AQwDl`
- Subtheme: `tbl17s0ll9r8VlVVw`

Only the question table was updated.

## Source → target field mapping

- `Oppgave` → `Oppgave (EN)`
- `Forklarende tekst` → `Forklarende tekst (EN)`
- `Riktig svaralternativ` → `Riktig svaralternativ (EN)`
- `Feil svaralternativ 1` → `Feil svaralternativ 1 (EN)`
- `Feil svaralternativ 2` → `Feil svaralternativ 2 (EN)`
- `Feil svaralternativ 3` → `Feil svaralternativ 3 (EN)`

Context fields that may be read but normally must not be changed:

- `ID`
- `Klasse`
- `Tema`
- `Undertema`
- `Tema (EN)`
- `Undertema (EN)`
- `Tags`
- `Bilde`
- `Har bilde`

## Norwegian traffic terminology for English Teoria content

Use clear British/international English for learners taking the Norwegian theory test in English.

Preferred terms:

- `vikeplikt` → `give way`, not `yield`
- `førerkort` → `driving licence`, not `driver's license`
- `kjørebane` → `carriageway`
- `blindsonen` → `blind spot`
- `trafikant` → `road user`
- `gangfelt` → `pedestrian crossing`
- `fotgjenger` → `pedestrian`
- `forbikjøring` → `overtaking`
- `møteulykke` → `head-on collision`
- `vegskulder` → `hard shoulder` or `shoulder`, depending on context
- `vognkort` → usually `vehicle registration certificate`
- `øvelseskjøring` → `practice driving` unless a more formal phrase is needed
- `pliktmessig avhold` → `compulsory abstinence`
- `berusende eller bedøvende middel` → `intoxicating or sedative substance`
- Norwegian `0,2 promille` → `0.02% BAC`, not `0.2 per mille`.
- When talking about the Norwegian drink-driving rule/threshold, prefer `blood alcohol limit` because Statens vegvesen uses this formulation in English and gives the limit as `0.02 percent`.
- Use percentage BAC values for promille/per-mille content: `0,2 promille` → `0.02% BAC`, `0,5 promille` → `0.05% BAC`, `1,5 promille` → `0.15% BAC`, `0,15 promille per hour` → `0.015% BAC per hour`.

Content rules:

- Preserve Norwegian legal/regulatory references; do not localize rules to the UK/US.
- Do not translate `Statens vegvesen` unless explanation is explicitly needed.
- Keep wrong alternatives plausible; do not make them more obviously wrong than the Norwegian source.
- Preserve intentional ambiguity and pedagogical precision.
- Keep explanations concise enough for product UI.

## Successful job pattern

For the Teoria QA job, the safe sequence was:

1. Fetch schema and verify every required field name, especially `Klar til prod`.
2. Fetch all records with `{Klar til prod}=TRUE()`.
3. Count filtered records and missing EN fields before translation.
4. Confirm expected missing row count from user when available.
5. Save backup of all filtered records and candidate records locally.
6. Generate a combined patch candidate file with only the six allowed EN fields.
7. Validate no candidate would overwrite non-empty EN fields in the backup.
8. Fresh-read every candidate immediately before patch and abort on non-empty target fields.
9. PATCH in batches of 10 records maximum.
10. Read back all filtered records and count remaining missing EN fields.
11. Write report and patch logs; upload summary artifacts to Google Drive when configured.

## Example result from the reference job

- Filtered `Klar til prod=true` records reviewed: 3881
- Records needing English backfill: 66
- Target fields per record: 6
- Total fields patched: 396
- Remaining missing EN fields after read-back: 0

Do not treat these counts as permanent facts. Use them only as a pattern for sanity checks when the user provides an expected number.
