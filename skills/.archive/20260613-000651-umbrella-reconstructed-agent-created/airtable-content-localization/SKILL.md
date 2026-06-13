---
name: airtable-content-localization
description: "Localize Airtable content safely: read-only counts, backups, translation candidates, controlled PATCH batches, read-back verification, and audit logs."
version: 1.0.0
author: Hermes Agent
license: MIT
---

# Airtable Content Localization (Archived)

This narrow localization/backfill skill was consolidated into `airtable` under "Content Localization / Backfill Workflow".

Core preserved workflow: inspect schema, count candidates read-only, create a local job directory, generate target-language candidates, validate JSON locally, do a fresh preflight read, PATCH only allowed target fields in controlled batches, then read back every changed record and keep an audit log. Never overwrite existing localized fields unless requested and never mutate source-language fields.
