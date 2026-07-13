---
name: spec
description: >
  Front-load a task spec before handing off to an implementation agent: scout scope, surface unknowns, draft checkable acceptance criteria, set review depth. Use BEFORE starting any non-trivial ticket/feature — when about to tell an agent to "implement X" or "do this TDD style". Kills mid-session re-exploration; output is the handoff prompt.
---

Build the spec up front so the implementation agent doesn't rediscover scope mid-session. Two gates, both load-bearing: user **drives the load-bearing calls** (via `/map`) before drafting, user corrects draft before handoff. Don't collapse them.

## Input

`/spec <linear-id | one-line task>`. Multiple related tickets → one shared-subsystem spec (batch them; reuse map context).

## Phase 1 — Map + decide (STOP at end, do NOT draft)

Run the `map` skill on the ticket/task — do NOT present a "proposed approach" for the user to rubber-stamp. `/map`:
1. Fetches the ticket (Linear MCP) if ID given; too vague → STOP, ask first.
2. Fans out read-only explore subagents → territory as `path:line` anchors (scope), and the **decisions the task forces**, triaged trivial vs load-bearing.
3. **Withholds AI's answer on load-bearing decisions until the user commits theirs** (prediction-first), then reveals + refutes each call.

Output of Phase 1 = the user's **decided calls** + scope anchors. The approach *emerges from the user's calls*, not from an AI proposal they approve. Mis-scoped here = cheap; mid-impl = not.

### High-stakes design forks (judge panel)

A load-bearing `/map` decision that's a genuine irreversible fork — wide blast radius, or the user is circling without converging — feed their call with candidates: generate 2–3 independent approaches and score them with a **clean-context judge panel** (separate agents, each scoring blind to the others', on correctness / blast radius / reversibility / complexity). Surface the winner + the losing trade-offs so the **user decides** — this informs their `/map` call, it doesn't replace it. Gate by stakes — skip routine/reversible. Heavy fan-out → the Workflow tool's "judge panel" pattern.

## Phase 2 — Draft

After the user's `/map` calls are committed, write the spec from THEM (the approach = their decided calls, not an AI proposal):

- **Goal** — 1 line.
- **Acceptance criteria** — checkable boxes, behavioral, each paired 1:1 with how it's verified. Not "works".
- **Scope** — files/endpoints to touch (`/map` anchors) + reference pattern(s) to mirror (`path:line`).
- **Decided calls** — the load-bearing decisions + the user's ruling on each. Any `SKIP:` → carried here as an explicit un-owned decision.
- **Out of scope** — deferred/adjacent work → tickets (surface-and-park), not silent drops.
- **Constraints / don't-break** — shared callers, auth boundaries, formats to preserve.
- **Test plan** — failing test per AC first (TDD); for a bug, the repro. Load `tdd`.
- **Calibration** — `DEEP REVIEW` (read diff) vs `OUTCOME-ONLY` (verify green). Shared/critical path or subtle-wrong-expensive → deep; one-shot/mechanical/scripted → outcome. Depth scales with blast-radius + unfamiliarity, NOT describability — the well-understood high-stakes change is the one you're tempted to hand off thin, and the one that most needs depth.
- **Refute before handoff (high-stakes)** — shared/irreversible/critical-path or multi-PR decomposition: fresh-context adversarial pass on the drafted spec — agent given only ticket + spec, tasked to find the fatal flaw (unstated assumption, un-enumerated data-path consumer, mis-sequenced/oversized slice, AC with no verification step). Per Planning rule. Distinct from Phase-1's judge panel (fork selection) — this refutes the chosen plan. Skip routine/reversible.

Present draft. User corrects — this is the judgment step, don't skip it.

## Phase 3 — Handoff

On approval:

1. Save to `.dave-ai-tasks/<TICKET>.md` (ID, or a slug from the task if no ID; create dir if absent). Match existing file convention.
2. Use the approved spec as the implementation agent's opening prompt — context already loaded, no re-exploration.
3. Fan out to parallel agents ONLY if partitions are independently reviewable with no cross-partition contract coupling (file-disjoint ≠ independent) — and confirm that split separately, don't fold it into the spec approval.

## Rules

- Never draft before the user's `/map` calls are committed. Never hand off before Phase 2 correction. The gates ARE the skill.
- Spec fits on a screen. Overflows → scope too big, split it.
- Don't pad. Fragments over sentences.
