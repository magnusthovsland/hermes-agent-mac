---
name: media-content-workflows
description: "Umbrella workflow for media tasks: YouTube transcript reuse, GIF search/download, audio analysis, songwriting prompts, and AI music generation."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [media, youtube, gif, audio, music, songwriting, transcripts]
---

# Media Content Workflows

Use this skill when the user asks to find, transform, analyze, summarize, or generate media-adjacent content: YouTube transcripts, GIFs, audio feature visualizations, lyrics, song prompts, or AI music generation.

## Choose the path

- **YouTube/video-to-text** → fetch transcript first, then produce the requested output format (summary, thread, blog, notes, critique).
- **GIF search/download** → use Tenor-style search/download, verify file path/URL, and keep query terms safe and specific.
- **Audio inspection** → compute spectrograms/features (mel, chroma, MFCC) and report what was actually generated.
- **Songwriting** → craft lyric structure, prosody, rhyme, emotional arc, and genre tags before model prompting.
- **AI music generation** → prepare lyrics + tags + provider-specific constraints; verify returned files/URLs.

## Workflow

1. Identify whether the deliverable is a file, a prompt, an analysis, or a transformed text artifact.
2. Retrieve or create the source material with tools; do not invent transcripts, downloads, or generated media.
3. Preserve reusable commands and output recipes in references/scripts.
4. Verify the artifact exists or the API returned a real handle.
5. Final response should include path/URL plus a concise description of content and any limitations.

## Labeled playbooks

### YouTube content

Fetch transcripts with the preserved script under `references/packages/youtube-content/scripts/fetch_transcript.py`, then format into summaries, blog posts, threads, timestamps, or study notes using the output-format reference.

### GIF search

Use short descriptive search terms, inspect results before choosing, and download the actual GIF when the user needs a file.

### Audio features

Use songsee-style CLI workflows for spectrogram and feature extraction. Mention sample rate, duration, and feature type when reporting.

### Songwriting and AI music

Separate craft from generation: first write/refine lyrics and tags, then call the generation provider or prepare the exact prompt. HeartMuLa-specific details are preserved under `references/packages/heartmula/`; general songwriting guidance is under `references/packages/songwriting-and-ai-music/`.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `gif-search`, `heartmula`, `songsee`, `songwriting-and-ai-music`, and `youtube-content`.
