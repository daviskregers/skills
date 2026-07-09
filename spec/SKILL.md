---
name: spec
description: >
  Front-load a task spec before handing off to an implementation agent: scout scope, surface unknowns, draft checkable acceptance criteria, set review depth. Use BEFORE starting any non-trivial ticket/feature — when about to tell an agent to "implement X" or "do this TDD style". Kills mid-session re-exploration; output is the handoff prompt.
---

Build the spec up front so the implementation agent doesn't rediscover scope mid-session. Two gates, both load-bearing: user confirms scope before drafting, user corrects draft before handoff. Don't collapse them.

## Input

`/spec <linear-id | one-line task>`. Multiple related tickets → one shared-subsystem spec (batch them; reuse scout context).

## Phase 1 — Scout (STOP at end, do NOT draft)

1. Fetch ticket (Linear MCP) if ID given. Extract goal + stated AC.
2. Fan out read-only explore subagents (parallel) to locate actual touchpoints: files, endpoints, call-sites, shared services the change rides on. Read-only — no edits in `/spec`.
3. Present, then STOP for confirmation:
   - **Scope found** — concrete files/endpoints.
   - **Open questions** — 2–5 things genuinely un-inferable from ticket+code.
   - **Proposed approach** — 1–2 lines.

Wait for user to confirm/correct direction. Mis-scoped here = cheap; mid-impl = not. This gate prevents rubber-stamping a wrong scope.

### High-stakes design forks (judge panel)

If the approach is a genuine fork — irreversible, wide blast radius, or you're circling without converging — don't present a single approach. Generate 2–3 independent candidates and score them with a **clean-context judge panel**: separate agents, each scoring the candidates blind to the others' verdicts, on explicit criteria (correctness, blast radius, reversibility, complexity). Recommend from the winner, grafting the best ideas from runners-up; surface the losing trade-offs so the user decides. Gate by stakes — skip for routine/reversible calls. Heavy fan-out → the Workflow tool's "judge panel" pattern.

## Phase 2 — Draft

After confirmation, write the spec:

- **Goal** — 1 line.
- **Acceptance criteria** — checkable boxes, behavioral. Not "works".
- **Scope** — files/endpoints to touch (Phase 1, corrected).
- **Constraints / don't-break** — shared callers, auth boundaries, formats to preserve.
- **Calibration** — `DEEP REVIEW` (read the diff) vs `OUTCOME-ONLY` (verify green, don't read every line). Touches shared/critical path, or subtle-wrong is expensive → deep. One-shot / mechanical / scripted → outcome.

Present draft. User corrects — this is the judgment step, don't skip it.

## Phase 3 — Handoff

On approval:

1. Save to `.dave-ai-tasks/<TICKET>.md` (create dir if absent). Match existing file convention.
2. Use the approved spec as the implementation agent's opening prompt — context already loaded, no re-exploration.
3. If batched and the work partitions into independent file-scoped chunks → hand partitions to parallel agents.

## Rules

- Never draft before Phase 1 confirmation. Never hand off before Phase 2 correction. The gates ARE the skill.
- Spec fits on a screen. Overflows → scope too big, split it.
- Don't pad. Fragments over sentences.
