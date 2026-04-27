---
name: artifact-output
description: >
  Convention for saving generated artifacts (reviews, explanations, reports) to
  .ai-artifacts/ directory. Output file path and one-line summary in chat only —
  never dump artifact content. Use when an agent or command produces output that
  the user reads in their editor, not in Claude Code.
---

Save generated artifacts to `.ai-artifacts/` directory. User reads them in their editor — not in chat.

## Saving

- Directory: `.ai-artifacts/` relative to project root. Create if needed.
- Filename: `<type>_<YYYY-MM-DD>_<HH-MM-SS>[_<suffix>].<ext>`
  - Type: `review`, `explanation`, etc.
  - Timestamp: UTC
  - Suffix: optional slug or random chars for dedup
  - Extension: `.md` for reviews, `.html` for explanations, etc.

## Chat Output

**NEVER output artifact content in chat.** Chat output MUST be ONLY:

1. File path
2. One-line summary (under 20 words)

No code snippets, no detailed findings, no sections from the artifact. The file path IS the deliverable.

## Why

User opens artifact in editor where they can: jump to file locations, search, copy blocks, see syntax highlighting. Chat output is throwaway — artifact file is the permanent record.
