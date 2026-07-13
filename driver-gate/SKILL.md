---
name: driver-gate
description: >
  Shared anti-rubber-stamp pattern for any present-then-approve workflow — planning, code review, PR-comment triage, finding triage, verification. Keeps YOU the decider on load-bearing calls by withholding AI's verdict/answer until you commit yours WITH A RATIONALE (prediction-first), triaging trivial-vs-load-bearing so only real judgment calls demand you, and shaping friction so deciding is cheaper than skipping. Invoked by /map, /comment, /comments, /spec, /ship, /probe. Load this whenever a workflow would otherwise show you a conclusion and ask for a thumbs-up.
---

The problem this kills: any workflow that **presents a conclusion and asks approval** lets you take the cheap exit — "fix" / thumbs-up — instead of the expensive one (understand). Reviewing a conclusion is reviewing a *map*; the errors live in the *territory*. Your judgment fires when you build the model yourself, not when you approve someone else's. This pattern recreates that deliberately, and stakes-tiers it so it stays affordable.

## The four mechanics (all workflows apply these)

1. **Territory, not conclusions.** Show the source — `path:line` anchors the user can jump to + the *question* — not a pre-digested verdict/plan/summary. **Anchor neutrality (or the verdict leaks):** annotate each anchor with what factually *exists* there (columns present, indexes present, the call made), never editorialize absence — "no unique index" / "missing constraint" telegraphs the answer. Enumerate ALL touchpoints, not the subset that points one way, so the user derives the gap themselves rather than being pointed at it. (You can't fully hide the answer from a sharp reader — that's fine; the goal is that THEY infer it from facts, not that YOU state it.) Don't conclude what they should read for themselves.
2. **Triage — shrink to what needs them.** Classify each item:
   - **trivial/mechanical** → AI handles it, records its default, no gate (typo, rename, obvious nitpick, reversible one-liner).
   - **load-bearing** → needs the user: irreversible / wide blast radius / defines logic correctness / crosses layers / touches security or data integrity / the class of thing they'd have caught by hand.
   Over-gating trains bypass; under-gating hands back the stamp. **>~3 gates surfacing is a SIGNAL the task is too big — split it first.** If it's genuinely irreducible, gate them all but flag the task is heavy; never DROP a real load-bearing gate just to hit a number (that's under-gating by fiat). If everything triages trivial, say so, don't manufacture gates. AI owns triage, so it must justify each gate in one phrase — an unjustifiable gate is over-gating.
3. **Prediction-first — commit a *rationale*, not a token.** On load-bearing items, **WITHHOLD AI's answer/verdict/recommendation until the user commits theirs — and the commit must include a one-line *why* grounded in the territory** ("service layer, because the controller can't see concurrent writes"), not a bare verdict word. A rationale can't be bluffed the way "yes"/"db" can; requiring it is what stops prediction-first from laundering a guess as ownership. Genuinely can't form a rationale → that's the signal to build the model first (route to `explain`/`tutor`), not to guess. **Then reveal + challenge symmetrically:** attack AI's OWN answer as hard as the user's, and treat **the user's pre-reveal call as binding** unless they actively overturn it with a stated reason — the reveal informs, it does not get to quietly become the decision. Dig where the two diverge.
4. **Friction asymmetry.** Point friction the right way:
   - **Wanted = zero effort:** auto-open the artifact in their editor (`tmux new-window -n <n> "nvim <path>"`, fall back to printing path), anchors jump-ready. Engaging is the default action.
   - **Unwanted = tedious + visible:** a load-bearing item left blank, answered "fix"/"you decide"/deferral, or given a bare verdict with no rationale does NOT proceed. To skip, they type `SKIP: <reason>` per item — deciding-with-a-why is *less* effort than justifying not-deciding. Every `SKIP:` is carried verbatim into downstream output (spec / PR / commit) as an **"un-owned decision"** — the stamp leaves a mark they and reviewers see.
   Never let skipping be cheaper than deciding.

## Application shapes

**A. Decision scout** (planning / design) — `/map`. Surface the forks a task forces + territory anchors, withhold AI's answers, user decides each load-bearing call, then reveal + refute. See the `map` skill.

**B. Finding / comment triage loop** — `/comment` (single item), `/comments` (PR comments), `/ship` (Phase A local-review findings, Phase D PR comments). Over a queue of findings/comments, one at a time (NEVER batch verdicts):
1. Header `N/total — path:line — source`. Quote the item verbatim (comment body / finding text) so the user sees the original.
2. **Triage.** Trivial → say so, handle fast (still show what you did). Load-bearing → gate:
3. **Territory + withhold.** Show the referenced code as `path:line` anchors + the question ("is this real, and why?"). **Do NOT state your verdict yet.**
4. **User commits their read + why** — Real / False positive / Debatable, with a one-line rationale from the code. Bare verdict, no rationale → doesn't proceed. Skip only via `SKIP: <reason>`.
5. **Reveal + challenge symmetrically.** Now give AI's verdict + reasoning. Attack AI's own verdict as hard as the user's; the user's pre-reveal call binds unless they overturn it with a stated reason. Diverge → dig there.
6. **Fix on confirm** — TDD (failing test → minimal fix → suite). Load `tdd`.
Carry any `SKIP:` into the close note / PR reply as an un-owned decision.

**C. Verification** (real-behavior checks) — `/probe`, `/ship` Phase A2. AI eats setup friction (env, seed, fixtures, ready `.http`/deep-link) but **withholds the checkmark**: the user predicts the expected result, runs the probe, observes actual, and rules match/mismatch — AI never self-certifies. Same withholding as A/B, applied to the green box. See the `probe` skill.

## Calibration

- Depth scales with blast-radius + unfamiliarity, NOT how easy the item is to describe. The well-understood high-stakes call is the one you're tempted to wave through.
- Triage honestly every run — the split is the affordability lever. A queue of 30 Copilot nitpicks with 2 real concerns should gate 2, not 30.
- Anchors over prose. Verdicts terse. Code as `path:line`, never dumps.
- **Honest limit:** these mechanics are self-enforced by the AI in a single turn — it has already computed the answer it's "withholding." Prompt discipline reduces the leak but doesn't guarantee it; the real teeth is an out-of-band **hook** (cf. `tdd-reminder`) that nudges/blocks when a load-bearing edit or `--ready` lands with no committed rationale. Treat the hook as the enforcement layer, this skill as the protocol.
- The goal is a built model, not a filled form. If the user can't give a territory-grounded rationale, prediction-first has found the gap — route to `explain`/`tutor` to build the model; don't accept a guess.
