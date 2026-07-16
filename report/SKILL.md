---
name: report
description: >
  Research a topic and produce a self-contained, dark-themed HTML report, then self-review it.
  Fans out parallel research agents, writes from a fixed template (scientific [n] citations with
  hover→quote→source, glossary hovercards for people/companies/tech, in-memory annotation tool with
  edit, Mermaid diagrams with click-to-zoom/pan + persistent legend), then runs an adversarial review
  for missing context, representation bias, formatting conformance, readability, and citation integrity.
  Use when asked to research something and produce a shareable report/briefing. For code-explanation
  HTML use `/explain` (which reuses this scaffold); for a runtime-behaviour check use `/probe`.
---

Turn a topic into a researched, cited, reviewable HTML report. Template: `assets/report-template.html` (this skill's dir) — copy it, never hand-roll the scaffold. Load `diagram` (Mermaid conventions) + `artifact-output` (save location).

## Input

`/report <topic | question>` (+ optional `--deep` for a wider fan-out). **Underspecified → STOP and ask 2–3 clarifying questions** (scope, audience, angle, region/time bound) before researching — a vague topic yields a vague report. Weave the answers into the research questions.

## Phase 1 — Research (fan out)

1. Decompose the topic into 4–8 independent sub-questions / angles (dimensions, opposing views, named examples, counter-evidence). Deliberately include an angle that could *disconfirm* the premise.
2. Fan out one read-only research agent per angle (Agent tool: `Explore` / `general-purpose`, or `WebSearch`/`WebFetch` for web topics) — parallel, in one message. Each returns claims and **appends every source it actually fetched** as a line to a shared `.dk-notes/reports/<slug>.sources.jsonl`: `{url, fetched: true|false, quote: "<verbatim span>", date, tag: "independent"|"vendor"}`. Scale width to `--deep` / topic size; `log` what you dropped if you cap.
3. **Anti-fabrication — mechanical, not self-attested** (the gate has teeth only if a later step diffs against an artifact):
   - Phase 2 may cite **only** URLs present in `<slug>.sources.jsonl`. A claim with no backing line → cut it, or mark it inference (cap + track inference-tagged claims; a report leaning on them is flagged thin).
   - A quote shown in a hovercard / `#:~:text=` deep-link must be the verbatim `quote` from that source's jsonl line. **Never invent a URL, quote, or statistic** — not in the jsonl → doesn't ship.
   - Note where the *only* backing is self-interested (vendors selling the thing) — surface it in Phase 3, don't hide it.

## Phase 2 — Write

Copy the template to `.dk-notes/reports/<slug>.html` and fill the AUTHOR blocks:
- **Sections** — lead + `h2` sections; stat rows for headline numbers; `callout`/`callout warn` for caveats and counter-currents.
- **Every non-obvious claim gets an inline `[n]`** citation. Numbers must be contiguous and each map 1:1 to a `#refN` row.
- **References** — one row per source: real URL (deep-linked with `#:~:text=<quote>` where a quote exists), `independent`/`vendor` pill, what it backs, and a **date**. No dangling or invented refs.
- **Glossary** — add every person/company/technology a general reader might not know to the `GLOSSARY` object; occurrences auto-link.
- **Diagrams** — only where there's real flow (per `diagram`); put a `<div class="legend">` immediately before each `<div class="mermaid">` so it shows in the zoom overlay.
- Footer keeps `🤖 Generated with AI`.
- **Exit check (run it, paste the output):** (a) extract all `href="#refN"` and all `id="refN"` — assert the two sets are equal and contiguous `1..N` (no skipped / dangling / duplicate); (b) assert every `#refN` URL appears in `<slug>.sources.jsonl`. A failure blocks Phase 3.
- Report the path per `artifact-output`; don't dump HTML in chat.

## Phase 3 — Review (fresh-context, adversarial)

Spawn a review agent (clean context — give it only the rendered report + the topic) to check the four dimensions; fix confirmed findings, then re-open:

1. **Missing context / balance** — are quantified claims backed by the right evidence (e.g. an ROI/productivity claim for a *public* company → check for financials, not just a blog)? Is there **representation bias** (over-weighting parties who profit from the narrative — model vendors, hyperscalers)? Does it cover not just *how* something is used but **whether it works** — outcomes, user/customer satisfaction, scandals or counter-evidence (e.g. backlash, walk-backs)? Name what's under-represented.
2. **Formatting conformance** — dark default (+ light toggle), citations with working hovercards→quote→source, glossary hovercards, annotation tool present, Mermaid diagrams zoomable with legend, references contiguous, AI attribution present.
3. **Readability for a non-expert** — jargon defined (glossary), plain lead, digestible chunks, no wall-of-text.
4. **Citation integrity** (hard gate — give the review agent `WebFetch`) — diff **every** `#refN` URL against `<slug>.sources.jsonl` (any ref not present = fabricated → fail); then fetch a sample and string-match the `#:~:text=` quote against the live page text, and confirm the quote actually supports the claim it's attached to. Any fabricated or mismatched source fails the report — fix before shipping. The review agent's transcript (which refs it fetched + pass/fail) is the proof it ran — no transcript, not done.

Then **automatically open the report in the browser** — don't make the user ask (`open <path>` on macOS, `xdg-open` on Linux). Re-open the same way after applying any annotation-round edits. State honestly what you verified and what's still thin.

## Notes

- Annotations are **in-memory** (cleared on refresh) by design — they're for a review pass, copied back into chat, not persisted.
- Mermaid + svg-pan-zoom load from CDN → diagrams need internet; text/citations/glossary/annotations work offline.
- Iterate via annotations: the reader marks up the report, copies the annotations, pastes them back — you apply them and re-open.
- Dual-setup: this skill lives in the shared skills submodule — sync per the CLAUDE.md dual-setup rule.
