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

Only load-bearing decisions become prediction gates. Everything else moves fast. If everything triages trivial, say so — don't manufacture gates. **>~3 gates = signal to split the task** (driver-gate mechanic 2); if genuinely irreducible, gate all but flag it's heavy — never drop a real gate to hit a number.

## Phase 3 — Emit map + STOP (answers WITHHELD)

Write scratch markdown to `.dave-ai-tasks/<TICKET>.map.md` (ID, or slug if none; create dir if absent). Structure:

- **Per load-bearing decision** — grep-style anchors (`path:line` + one **neutral factual** line of what exists there — never "no X"/"missing", which telegraph the answer), the question, and `→ your call + why: [ ]`. **Do NOT write your own answer.** The point is you commit — with a reason from the code — before you see AI's.
- **Trivial decisions** — one line each + AI's chosen default (transparency, not a gate).
- **Raw territory** — relevant anchors not tied to a decision, for context. Enumerate the touchpoints; don't hand-pick the subset that points one way.
- **Prompt** — "Add any decision I didn't flag." Explicitly invite the miss — AI's grouping is a menu, not the ceiling.

Example block (anchors state what IS, not what's absent):
```
## Decision: which layer owns org uniqueness?  [load-bearing]
current enforcement:
  app/Http/Controllers/OrgController.php:88   validateUnique() called inline
  app/Models/Organisation.php:34              fillable: name, slug; casts only
  database/migrations/..._orgs.php:12         columns: id,name,slug; indexes: id
depends on this:
  app/Services/LoginResolver.php:51           reads Organisation::where(slug)
question: which layer should own uniqueness, and why?
→ your call + why: [ ]
```

**Auto-open (wanted path = zero friction):** after writing the file, drop the user straight into it — `tmux new-window -n map "nvim <path>"` (or `split-window`). Engaging the territory is the default action, not an opt-in. If tmux isn't present / command fails, fall back to printing the path. Don't claim it opened if it didn't.

Then STOP. User opens anchors in their editor, builds the model, fills each `→ your call + why`. Can't form a *why* from the code → that's the signal to build the model first (`explain`/`tutor`), not to guess. Do NOT proceed or reveal AI's opinion until calls are filled.

**Friction asymmetry (unwanted path = tedious + visible):**
- A load-bearing `→ your call` left blank, filled with "fix"/"you decide"/deferral, or given a bare verdict with no *why*, does NOT let the flow proceed. To skip, the user must type `SKIP: <reason>` on that line — deciding-with-a-why is *less* effort than justifying not-deciding.
- Every `SKIP:` is carried forward verbatim into downstream output (spec / PR) as an explicit **"un-owned decision"** line. The rubber-stamp leaves a visible mark for the user and reviewers — it stops being free.

## Phase 4 — Reveal + challenge (after user fills)

1. **Now** reveal AI's own answer + reasoning per load-bearing decision. Where it diverges from the user's call → dig there: that gap is the user's judgment firing (or a real risk on one side). Don't smooth it over.
2. **Refute symmetrically** — clean-context adversarial pass on BOTH calls: what breaks if the user's call is wrong (un-enumerated consumer, contract mismatch, missed edge) AND what breaks if AI's is. Attack AI's answer as hard as the user's — don't let the reveal quietly become the decision.
3. **The user's pre-reveal call binds** unless they overturn it with a stated reason. Output the decided calls — ready to feed planning. When invoked by `/spec`, these become Phase-2 draft input.

## Rules

- NEVER reveal AI's answer on a load-bearing decision before the user commits theirs. That withholding IS the skill — it removes the thumbs-up exit.
- NEVER propose a full approach/plan. This scouts territory and forces decisions; planning is downstream (`/spec`).
- Anchors, not prose summaries — `path:line` the user can jump to. Don't conclude what they should read for themselves.
- Triage honestly — over-gating trains the user to skip the gate. Under-gating hands back the rubber-stamp. The load-bearing set should be small and real.
- Read-only. No edits, no writes outside the `.map.md` scratch file.
- Friction points one way: wanted path frictionless (auto-open, jump-ready anchors), unwanted path costs keystrokes (`SKIP: <reason>`) + a visible mark. Never let skipping be cheaper than deciding.
