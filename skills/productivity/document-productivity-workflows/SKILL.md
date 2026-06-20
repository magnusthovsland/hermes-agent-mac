---
name: document-productivity-workflows
description: "Umbrella workflow for document-centric productivity: OCR/extraction, PDF edits, PowerPoint decks, and office file manipulation."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [documents, pdf, ocr, powerpoint, office, productivity]
---

# Document Productivity Workflows

Use this skill for document-centric work: extracting text from PDFs/scans, editing PDFs, creating or modifying PowerPoint decks, and manipulating Office files.

## Choose the path

- **Need text from a PDF/scan** → use OCR/document extraction first (`pymupdf` for text PDFs, marker/OCR paths for scanned or layout-heavy documents).
- **Need a small PDF text/title correction** → use nano-pdf-style natural-language edits, then verify the resulting PDF.
- **Need a presentation** → create/read/edit `.pptx`, slides, notes, and templates with PowerPoint scripts; verify the deck structure and, when possible, render thumbnails.
- **Need generic Office package work** → inspect/unpack the OOXML package carefully; preserve relationships and content types.

## Operating rules

1. Keep originals unchanged; write outputs with explicit new filenames unless the user requests overwrite.
2. Verify real artifacts: file exists, page/slide count is plausible, extracted text is non-empty, or modified PDF/deck opens structurally.
3. For scanned documents, report OCR uncertainty and any pages that failed.
4. For decks, prefer programmatic generation/editing over manual XML surgery unless the task requires low-level OOXML changes.
5. Include final paths and the exact command/script used.

## Labeled playbooks

### OCR and extraction

Use the preserved `ocr-and-documents` scripts under `references/packages/ocr-and-documents/scripts/` for PyMuPDF or marker extraction. Choose based on whether the PDF has embedded text or scanned pages.

### PDF text edits

Use nano-pdf recipes for targeted typo/title edits. Validate output and avoid claiming perfect layout preservation without inspection.

### PowerPoint

Use the preserved PowerPoint package under `references/packages/powerpoint/`, including scripts and Office schemas, for deck creation and OOXML-safe manipulation.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `nano-pdf`, `ocr-and-documents`, and `powerpoint`.
