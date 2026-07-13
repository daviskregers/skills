---
name: map
description: >
  Territory-first scouting that keeps YOU the driver: AI surfaces the code (grep-style anchors) and the decisions a task forces, but withholds its own answers until you commit yours (prediction-first). Triages trivial-vs-load-bearing so only the calls that need your judgment demand it. Use BEFORE planning any non-trivial ticket/change — when you'd otherwise let an agent plan it and rubber-stamp the output. Kills the reviewer trap: you decide the load-bearing calls, AI does the labor. /spec calls this as its scout.
---

Loads `driver-gate` for the shared mechanics (territory-not-conclusions · triage · prediction-first · friction-asymmetry) — this skill is its **Decision-scout** shape. Below is the scout-specific procedure.

Flip reviewer→driver. AI feeds you territory + the forks; YOU make the load-bearing calls. The gate that matters: AI withholds its own answer on load-bearing decisions until you've committed yours — so "just say fix" / thumbs-up isn't an available exit. Triage keeps the volume affordable; you only drive what needs you.

Why this exists: reviewing a *plan* is reviewing a map — bugs live in the *territory*. Your judgment fires when you build the model yourself, not when you approve someone else's. This recreates that deliberately, at anchor level, without you reading the whole changeset raw.

## Input

`/map <linear-id | one-line task>`. Multiple related tickets → one shared map.

## Phase 1 — Scout territory (read-only, NO conclusions)

1. Fetch ticket (Linear MCP) if ID — MCP down/bad ID → proceed from user text, note gap. Extract goal + stated AC. Too vague to scout → STOP, ask first.
2. Fan out read-only explore subagents (parallel) → locate actual touchpoints: files, endpoints, call-sites, current enforcement points, shared services the change rides on. Read-only — no edits.
3. Identify the **decisions this task forces** — forks where a choice could genuinely go >1 way: layer/validation placement, data flow, new abstraction, migration order, contract changes, auth boundary. Do NOT answer them.

## Phase 2 — Triage (shrink to what needs you)

Classify each decision:
- **load-bearing** → needs YOU: irreversible / wide blast radius / defines logic correctness / crosses layers / touches security or data integrity / you'd have caught a past bug here.
- **trivial/mechanical** → AI decides, just records its default (you can override, but no gate).

Only load-bearing decisions become prediction gates. Everything else moves fast. If everything triages trivial, say so — don't manufacture gates.

## Phase 3 — Emit map + STOP (answers WITHHELD)

Write scratch markdown to `.dave-ai-tasks/<TICKET>.map.md` (ID, or slug if none; create dir if absent). Structure:

- **Per load-bearing decision** — grep-style anchors (`path:line` + one line of what's there), the question, and `→ your call: [ ]`. **Do NOT write your own answer.** The point is you commit before you see AI's.
- **Trivial decisions** — one line each + AI's chosen default (transparency, not a gate).
- **Raw territory** — relevant anchors not tied to a decision, for context.
- **Prompt** — "Add any decision I didn't flag." Explicitly invite the miss — AI's grouping is a menu, not the ceiling.

Example block:
```
## Decision: where is org uniqueness enforced?  [load-bearing]
current enforcement:
  app/Http/Controllers/OrgController.php:88   validateUnique() inline
  app/Models/Organisation.php:34              no constraint
  database/migrations/..._orgs.php:12         no unique index
depends on this:
  app/Services/LoginResolver.php:51           assumes controller guarantees it
question: which layer owns uniqueness, and why?
→ your call: [ ]
```

**Auto-open (wanted path = zero friction):** after writing the file, drop the user straight into it — `tmux new-window -n map "nvim <path>"` (or `split-window`). Engaging the territory is the default action, not an opt-in. If tmux isn't present / command fails, fall back to printing the path. Don't claim it opened if it didn't.

Then STOP. User opens anchors in their editor, builds the model, fills each `→ your call`. Do NOT proceed or reveal AI's opinion until calls are filled.

**Friction asymmetry (unwanted path = tedious + visible):**
- A load-bearing `→ your call` left blank, or filled with "fix"/"you decide"/deferral, does NOT let the flow proceed. To skip, the user must type `SKIP: <reason>` on that line — deciding is *less* effort than justifying not-deciding.
- Every `SKIP:` is carried forward verbatim into downstream output (spec / PR) as an explicit **"un-owned decision"** line. The rubber-stamp leaves a visible mark for the user and reviewers — it stops being free.

## Phase 4 — Reveal + challenge (after user fills)

1. **Now** reveal AI's own answer + reasoning per load-bearing decision. Where it diverges from the user's call → dig there: that gap is the user's judgment firing (or a real risk on one side). Don't smooth it over.
2. **Refute the user's calls** — clean-context adversarial pass: what breaks if each call is wrong? Un-enumerated consumer, contract mismatch, missed edge. This is where adversarial review belongs — on decisions the user OWNS, not on AI's plan.
3. Output the decided calls (user's, adjusted by the divergence discussion) — ready to feed planning. When invoked by `/spec`, these become Phase-2 draft input.

## Rules

- NEVER reveal AI's answer on a load-bearing decision before the user commits theirs. That withholding IS the skill — it removes the thumbs-up exit.
- NEVER propose a full approach/plan. This scouts territory and forces decisions; planning is downstream (`/spec`).
- Anchors, not prose summaries — `path:line` the user can jump to. Don't conclude what they should read for themselves.
- Triage honestly — over-gating trains the user to skip the gate. Under-gating hands back the rubber-stamp. The load-bearing set should be small and real.
- Read-only. No edits, no writes outside the `.map.md` scratch file.
- Friction points one way: wanted path frictionless (auto-open, jump-ready anchors), unwanted path costs keystrokes (`SKIP: <reason>`) + a visible mark. Never let skipping be cheaper than deciding.
