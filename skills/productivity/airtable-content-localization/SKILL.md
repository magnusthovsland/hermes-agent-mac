---
name: airtable-content-localization
description: "Localize Airtable content safely: read-only counts, backups, translation candidates, controlled PATCH batches, read-back verification, and audit logs."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  env_vars: [AIRTABLE_API_KEY]
metadata:
  hermes:
    tags: [Airtable, Localization, Translation, QA, Content, Audit]
    related_skills: [airtable, google-drive-deliverables]
---

# Airtable Content Localization

Use this skill when asked to fill, translate, localize, or backfill language-specific fields in Airtable records, especially when the table contains source-language fields and target-language fields such as `Question` → `Question (EN)`.

The goal is controlled content updates with strong auditability: **never overwrite existing localized content, never mutate source fields, always back up records before writing, and always read back after patching.**

## Trigger conditions

Load this skill for tasks like:

- Fill missing English fields in an Airtable QA table.
- Translate selected Airtable records from Norwegian to English.
- Backfill empty localized fields only where source fields exist.
- Produce an audit log of Airtable content changes.
- Compare how many production-ready records still lack target-language content.

## Safety rules

1. Verify the exact base ID, table ID, and field names from Airtable schema before writing.
2. Confirm the environment/branch/base: QA vs prod matters. If the user says QA, do not touch prod.
3. Filter records exactly as specified by the user; do not broaden the set silently.
4. Only update explicitly allowed target fields.
5. Never update source-language fields.
6. Never overwrite a target field that already has content.
7. Never update context fields such as category/topic/tag translations unless explicitly requested.
8. Before patching, do a fresh preflight read of every candidate record and abort if any target field is no longer empty.
9. Patch using `PATCH`, never `PUT`.
10. Batch Airtable writes in groups of at most 10 records.
11. Write JSON and CSV logs before and after patching.
12. Run read-back verification after patching.

## Recommended workflow

### 1. Schema verification

Use Airtable Metadata API to fetch the base schema and verify:

- table ID exists
- filter field exists
- all source fields exist
- all target fields exist
- context fields exist if needed

Save schema to the job directory.

### 2. Read-only count

Before generating translations, fetch all filtered records and compute:

- number of filtered records
- number of records with at least one empty target field
- missing count per target field
- count of records skipped because the source field is empty

If the user provided an expected count, use it as a hard sanity check before patching.

### 3. Create a job directory

Use a timestamped local directory:

```text
~/hermes-airtable-jobs/<task-slug>/YYYYMMDD-HHMMSS/
```

Recommended files:

- `schema.json`
- `filtered_records_backup.json`
- `translation_candidates_source.json`
- `translation_patch_candidates.json`
- `preflight_records_before_patch.json`
- `patch_log.json`
- `patch_log.csv`
- `readback_report.json`
- `report.md`

### 4. Generate target-language candidates

For each record:

- translate only fields whose target field is empty
- skip fields where source text is empty
- preserve the original meaning, degree of ambiguity, and answer plausibility
- keep target text concise enough for the product UI
- log uncertainty instead of guessing aggressively

### 5. Validate patch candidates locally

Before Airtable write, validate:

- candidate count matches expected count when one was supplied
- each candidate record ID exists in the source backup
- patch fields are a subset of the allowed target fields
- no candidate would overwrite an already non-empty target field from the source backup
- every requested target field has a non-empty proposed value

### 6. Fresh preflight read

Immediately before patching, fetch each target record again from Airtable. Abort if:

- the filter condition is no longer true
- any target field to be written now contains text
- record is missing or inaccessible

Save `preflight_records_before_patch.json`.

### 7. Patch

Use Airtable batch update endpoint with max 10 records per request:

```json
{
  "records": [
    {
      "id": "rec...",
      "fields": {
        "Target field": "Translated text"
      }
    }
  ]
}
```

Sleep briefly between batches to avoid rate-limit pressure.

### 8. Read-back verification

After patching, rerun the original filtered query and compute remaining missing target fields.

Report:

- records reviewed
- records updated
- fields updated per target field
- remaining missing records, grouped by reason
- paths to backup/log files
- uncertain IDs requiring manual review

## Translation quality notes

- Prefer domain-specific terminology supplied by the user or existing product content.
- Preserve legal/regulatory precision.
- Preserve deliberate ambiguity in wrong answer alternatives.
- Do not make wrong alternatives more obviously wrong than the source.
- Do not localize rules to another jurisdiction unless explicitly asked.
- Keep official Norwegian actor names when the product context requires it.
- For specialised regulatory/traffic terms, standardise wording across the whole filtered dataset after translation; if the user questions a term, verify current external usage and patch all affected records consistently, with a backup and read-back log.

## References

- `references/teoria-qa-english-backfill.md` — concrete Airtable field mapping, traffic terminology, and verification pattern from the Teoria QA English backfill job.
