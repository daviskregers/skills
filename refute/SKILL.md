---
name: refute
description: >
  Fresh-context adversarial review of any target — a plan/spec, a changeset/diff, files, a prompt/skill/config, a PR, or a bare claim. Fans out N independent refuters, each given ONLY the target + intent (no priming from the working conversation), each tasked to DISPROVE it, then synthesizes the findings that survive into a ranked list. Use when you want an honest "what's wrong with this" before committing/shipping/finalizing — the clean-context refutation the Planning + Verify rules call for. Distinct from /code-review (full local review), /spec's built-in refute (on a drafted spec), and /probe//verify (runtime behavior checks — refute is static adversarial *reading*, not execution).
---

Refute, don't rubber-stamp. The value is **clean context**: refuters that never saw the conversation can't inherit its assumptions. Each is told to break the target, default to "this is flawed" unless the target genuinely prevents the failure.

## Input

`/refute <target> [+ what it's meant to achieve]`. Target = file(s)/dir, a plan or spec (paste/path), a PR URL, a changeset (the working diff if unstated), or a claim. If the intent isn't obvious from the target, ask for it in one line — a refuter needs to know what "correct" means.

## Phase 1 — Frame (once)

State, in the prompt every refuter will get: (a) the target (paths to read / pasted content), (b) its INTENT in 1-2 lines, (c) nothing from this conversation's reasoning. Keep it clean — priming defeats the point.

## Phase 2 — Fan out refuters (parallel, clean context)

Spawn independent agents, each a **distinct lens** (diversity catches what redundancy can't). Pick lenses by target type:
- **plan / spec** → unstated assumption · un-enumerated data-path consumer · mis-sequenced or oversized slice · acceptance criterion with no verification step.
- **code / changeset** → correctness + edge cases · does-it-actually-work (trace a failing input) · enforceability (can the guard be bypassed) · regression/blast-radius.
- **prompt / skill / config** → enforceability (is the instruction actually binding) · self-defeat (does it backfire / train the wrong habit) · coherence + gaps (dangling refs, contradictions between files).
- **claim** → find the counterexample · is the evidence load-bearing or vibes.

Each refuter returns a ranked list: concrete failure (most severe first), WHY, fatal-vs-fixable + minimal fix. Default N=3 lenses; "thorough"/high-stakes → widen + add a completeness critic ("what modality/consumer/claim went unchecked?").

## Phase 3 — Synthesize (barrier — needs all findings)

Dedupe across refuters (same defect from two lenses = one finding, higher confidence). Rank most-severe first. **Cut findings that don't survive scrutiny** — a refuter overstating worst-case is itself refutable; note where you discounted one and why. Separate: fatal (blocks) vs fixable (patch) vs overstated (noted, dropped).

## Output

Ranked surviving findings, terse: each = defect + why + fatal/fixable + minimal fix. `path:line` not prose. Lead with the verdict (ship / fix-first / rethink) and the single biggest risk. Convergence across independent lenses → flag it (strong signal). Optionally save via `artifact-output` if the user wants it filed; otherwise chat only.

## Rules

- Refuters get CLEAN context — target + intent only, never this conversation's reasoning or conclusions.
- Distinct lenses, not N copies of the same skeptic — diversity is the point.
- Don't relay walls of refuter text — synthesize + compress (Brevity). Drop what didn't survive.
- Scale N to stakes: quick sanity → 2-3; "thorough"/shared/irreversible → widen + completeness critic.
- Be honest when a refuter is wrong — refute the refutation too; don't inflate the finding count.
