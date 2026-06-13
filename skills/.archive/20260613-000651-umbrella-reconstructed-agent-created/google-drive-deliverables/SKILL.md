---
name: google-drive-deliverables
description: Deliver reports, analyses, and Office/PDF artifacts to Google Drive folders with read-back verification.
version: 1.0.0
author: Hermes Agent
license: MIT
---

# Google Drive Deliverables (Archived)

This narrow Drive-delivery skill was consolidated into `google-workspace` under "Drive Deliverables Workflow".

Core preserved workflow: create the artifact locally first, choose the correct file format, upload with `$GAPI drive upload` to the intended folder, read back metadata, and report the verified link. Permission changes, deletion, overwrites, or sharing with new people require explicit confirmation.
