---
name: probe
description: >
  Low-friction real-behavior verification where AI eats ALL the setup friction and hands YOU a one-action check to run and judge. AI spins the env, seeds state, GENERATES the fixtures (XLSX/CSV/JSON/upload files with trigger + edge data), and builds a ready probe (kulala .http for APIs, deep-link URL + click-path for UI) — then YOU predict the expected result, fire it, observe, and rule. AI withholds its own expected until you've predicted; it never checks the box. Use to confirm a change actually works end-to-end when writing the fixture / finding the page / setting up state is why you'd otherwise skip it. The driver-gate pattern applied to verification. Pairs with /run (launch), kulala (.http), driver-gate.
---

The problem this kills: AI-written tests don't give you the confidence your own TDD did (you didn't encode the contract or witness red→green), so you reach for real-behavior checks — but the friction (don't know where in the browser, tedious setup, hand-crafting fixtures) makes skipping cheaper than checking, so you re-offload to `/verify` and rubber-stamp a green box. This flips the gradient: **AI removes every setup cost; you stay the observer.** Loads `driver-gate` (triage · prediction-first · friction-asymmetry).

## Input

`/probe [AC / ticket / diff scope]` — none? Derive eyes-on candidates from the working diff + any spec ACs.

## Phase 1 — Triage what needs eyes-on

From the ACs / diff, split (driver-gate mechanic 2):
- **Green-safe** → pure logic well-covered by tests; no probe. Say so, don't manufacture one.
- **Eyes-on** → real-behavior confirmation genuinely adds signal: integrations, uploads/imports, rendering, auth/permission boundaries, data migrations, anything that "passes tests but could be wrong in the running system."
Keep the eyes-on set small and real.

## Phase 2 — Prep (AI eats the friction)

Per eyes-on AC, do the tedious part so the user doesn't:
1. **Env** — ensure the app is running (invoke `run` skill / tilt). Report the URL/base.
2. **State** — seed the exact DB/app state the check needs (the org, the user, the permissions, the prior records).
3. **Fixtures** — GENERATE the input artifacts: the XLSX / CSV / JSON / upload file, seeded with data that (a) triggers the behavior under test and (b) includes the edges that matter (malformed row, boundary value, duplicate, empty). Save to a known path. **Falsifiability (guards a hollow probe):** the fixture must be designed to hit the changed `path:line` and would give a DIFFERENT result against the pre-change code — a probe that'd pass against the OLD code proves nothing. This is an internal design check: confirm it privately, but do NOT surface "why the result differs" to the user before they predict (that spoils Phase 3); reveal it in Phase 4. If env/seed/auth can't actually be set up, say so — don't hand over a probe that exercises nothing.
4. **Probe** — build the one-action check, medium chosen per-check:
   - **API-shaped** → a ready `kulala` `.http` file (auth token captured via a named request per `kulala` skill), request body / upload referencing the fixture path.
   - **UI-only** → a **deep-link URL** to the exact page/state + a one-line click-path. Don't make the user hunt.

## Phase 3 — User predicts FIRST (STOP; expected AND runnable probe both WITHHELD)

State that the probe is built and what each check exercises — the `.http`/fixture **path** and the **question** ("what should this return / show?"). **Withhold TWO things until the user commits a one-line prediction per check:** (a) AI's expected result (the checkmark in disguise), AND (b) the *runnable* probe itself. **Do NOT open/auto-run the `.http`, print a ready-to-paste command, or hand over the deep-link yet** — opening the fireable artifact before the prediction lets the user run it and read the result before committing, which silently defeats prediction-first (observed failure mode). Just name the path + ask. Can't predict → build the model first (`explain`/`tutor`), don't fire blind.

## Phase 4 — Open, user runs + judges, THEN AI reveals

0. **Only now** that predictions are locked, hand over the runnable probe: auto-open in the editor — `tmux new-window -n probe "nvim <path>"` (fall back to printing paths / the ready command).
1. User fires the probe, observes **actual**, and rules against **their own prediction** — match / mismatch.
2. **Only now** does AI reveal its own expected + reasoning. AI's ≠ user's prediction, or ≠ actual → dig there (a wrong expected that happens to match a wrong actual is the trap this ordering catches). The user's witnessing IS the confidence TDD used to give. **AI never self-certifies.**

## Phase 5 — Record (friction-asymmetry)

- **Match** → AC confirmed (the user owns it — they saw it).
- **Mismatch** → real bug; route to a TDD fix (failing test reproducing what they saw, then fix).
- **Skipped** → the AC is marked `UNVERIFIED` (via `SKIP: <reason>`), carried into the PR as an explicit unverified line — never a checked box. Skipping leaves a visible mark; running is one paste/click.
- **Open PR for this changeset?** (`gh pr view --json url`) → delegate a **Verification ledger** refresh to the `pr-describer` agent (never edit the description inline), one line per eyes-on AC: `✅ verified` (what was exercised) / `⚠️ UNVERIFIED: <reason>` / `N/A: <why>`. Pass the probe outcomes; it updates only the ledger section. No PR → skip.

## Rules

- AI never checks an eyes-on box on the user's behalf — it prepares, the user observes and rules. Withholding the checkmark IS the gate.
- Withhold AI's expected result until the user has predicted theirs (Phase 3). Handing over the expected turns the probe into a one-click confirm-AI's-claim — the exact rubber-stamp this kills.
- Withhold the *runnable* probe too — do NOT open the `.http` / print the ready command / hand the deep-link until the prediction is committed. Opening it first lets the user fire and read the actual before predicting, which defeats prediction-first just as surely as revealing the expected. Order: name-the-path + ask (Phase 3) → prediction locked → open + run (Phase 4).
- Every probe must be falsifiable: it should fail against the pre-change code. A probe that can't fail proves nothing — flag it instead of handing it over.
- Fixtures must be generated to trigger the behavior AND its edges, not just a happy-path stub — the edge rows are usually the point.
- Prefer CLI/`.http` over browser wherever the surface allows (user lives in neovim/tmux); deep-links only for genuinely visual checks.
- Triage honestly — probing green-safe logic trains the user to skip probes; skipping an eyes-on integration hands back the green-box rubber-stamp.
- Read/prepare-only until a confirmed mismatch → fix (TDD).
