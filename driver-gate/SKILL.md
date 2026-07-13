---
name: driver-gate
description: >
  Shared anti-rubber-stamp pattern for any present-then-approve workflow — planning, code review, PR-comment triage, finding triage. Keeps YOU the decider on load-bearing calls by withholding AI's verdict/answer until you commit yours (prediction-first), triaging trivial-vs-load-bearing so only real judgment calls demand you, and shaping friction so deciding is cheaper than skipping. Invoked by /map, /comment, /comments, /spec, /ship. Load this whenever a workflow would otherwise show you a conclusion and ask for a thumbs-up.
---

The problem this kills: any workflow that **presents a conclusion and asks approval** lets you take the cheap exit — "fix" / thumbs-up — instead of the expensive one (understand). Reviewing a conclusion is reviewing a *map*; the errors live in the *territory*. Your judgment fires when you build the model yourself, not when you approve someone else's. This pattern recreates that deliberately, and stakes-tiers it so it stays affordable.

## The four mechanics (all workflows apply these)

1. **Territory, not conclusions.** Show the source — `path:line` anchors the user can jump to + the *question* — not a pre-digested verdict/plan/summary. Don't conclude what they should read for themselves.
2. **Triage — shrink to what needs them.** Classify each item:
   - **trivial/mechanical** → AI handles it, records its default, no gate (typo, rename, obvious nitpick, reversible one-liner).
   - **load-bearing** → needs the user: irreversible / wide blast radius / defines logic correctness / crosses layers / touches security or data integrity / the class of thing they'd have caught by hand.
   Over-gating trains bypass; under-gating hands back the stamp. The load-bearing set must be **small and real** — if everything triages trivial, say so, don't manufacture gates.
3. **Prediction-first.** On load-bearing items, **WITHHOLD AI's answer/verdict/recommendation until the user commits theirs.** They can't rubber-stamp what they haven't independently assessed. Then reveal AI's take + reasoning, and **dig where the two diverge** — that gap is their judgment firing (or a real risk on one side). This withholding IS the pattern; without it the rest is theater.
4. **Friction asymmetry.** Point friction the right way:
   - **Wanted = zero effort:** auto-open the artifact in their editor (`tmux new-window -n <n> "nvim <path>"`, fall back to printing path), anchors jump-ready. Engaging is the default action.
   - **Unwanted = tedious + visible:** a load-bearing item left blank or answered "fix"/"you decide"/deferral does NOT proceed. To skip, they type `SKIP: <reason>` per item — deciding is *less* effort than justifying not-deciding. Every `SKIP:` is carried verbatim into downstream output (spec / PR / commit) as an **"un-owned decision"** — the stamp leaves a mark they and reviewers see.
   Never let skipping be cheaper than deciding.

## Application shapes

**A. Decision scout** (planning / design) — `/map`. Surface the forks a task forces + territory anchors, withhold AI's answers, user decides each load-bearing call, then reveal + refute. See the `map` skill.

**B. Finding / comment triage loop** — `/comment` (single item), `/comments` (PR comments), `/ship` (Phase A local-review findings, Phase D PR comments). Over a queue of findings/comments, one at a time (NEVER batch verdicts):
1. Header `N/total — path:line — source`. Quote the item verbatim (comment body / finding text) so the user sees the original.
2. **Triage.** Trivial → say so, handle fast (still show what you did). Load-bearing → gate:
3. **Territory + withhold.** Show the referenced code as `path:line` anchors + the question ("is this real, and why?"). **Do NOT state your verdict yet.**
4. **User commits their read** — Real / False positive / Debatable, in their words. Skip only via `SKIP: <reason>`.
5. **Reveal + challenge.** Now give AI's verdict + reasoning. Diverges from the user's? Dig there. Then adversarially test the surviving call (what breaks if it's wrong).
6. **Fix on confirm** — TDD (failing test → minimal fix → suite). Load `tdd`.
Carry any `SKIP:` into the close note / PR reply as an un-owned decision.

## Calibration

- Depth scales with blast-radius + unfamiliarity, NOT how easy the item is to describe. The well-understood high-stakes call is the one you're tempted to wave through.
- Triage honestly every run — the split is the affordability lever. A queue of 30 Copilot nitpicks with 2 real concerns should gate 2, not 30.
- Anchors over prose. Verdicts terse. Code as `path:line`, never dumps.
