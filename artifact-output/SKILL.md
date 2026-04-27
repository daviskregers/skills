---
name: artifact-output
description: >
  Save generated artifacts (reviews, explanations, reports) to .ai-artifacts/.
  Output file path + one-line summary in chat only — never dump content.
---

Save artifacts to `.ai-artifacts/`. User reads in editor, not chat.

## Saving

- Dir: `.ai-artifacts/` relative to project root. Create if needed.
- Filename: `<type>_<YYYY-MM-DD>_<HH-MM-SS>[_<suffix>].<ext>`
- Type: `review`, `explanation`, etc. Ext: `.md`, `.html`, etc.

## Chat Output

**NEVER output artifact content in chat.** ONLY:
1. File path
2. One-line summary (≤20 words)

No snippets, no findings, no sections. File path IS deliverable.
