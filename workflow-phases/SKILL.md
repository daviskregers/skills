---
name: workflow-phases
description: >
  Phased dev workflow — outer loop EXPLORE/PLAN/TEST/BUILD/CLEANUP/VERIFY around TDD's RED/GREEN/REFACTOR inner.
  Use when starting any non-trivial change: refactor, migration, rewrite, new feature, complex bug fix,
  or any task where "where do I start" is unclear. Separates understanding, planning, testing,
  building, cleanup, verification — with backtracking when later phases reveal earlier mistakes.
  Skip for trivial single-line changes or fully-understood code.
---

Dev workflow outer loop. 6 phases. TDD inner loop (RED/GREEN/REFACTOR) spins inside TEST+BUILD — see `tdd` skill.

Order: 🔍 EXPLORE → 📋 PLAN → 🔴 TEST → 🟢 BUILD → 🧹 CLEANUP → ✅ VERIFY.

Wheel ≠ pipeline — any phase can backtrack to earlier one when later reveals an earlier mistake. Skip wheel for trivial / fully-understood / throwaway work.

## 🔍 EXPLORE — understand before changing

- **Goal**: map code, callers, data flow, adjacent surfaces, test gaps. Knowledge, not code.
- **Output**: map artifact — file list, callers, reads/writes, adjacent surfaces, test coverage, open questions, proposed slicing. Save to `.dk-notes/exploration/<slug>.md`.
- **Exit**: can answer (1) what changes, (2) what's at risk, (3) how to slice.
- **Forbidden**: edits to source, committing impl. Read-only.
- **Time-box**: 30-60 min spike. Blow box → area more tangled than expected, replan.
- **Skipped signal**: scope discovered mid-BUILD, stale-read bugs after merge, "didn't know X existed."

## 📋 PLAN — slice into PRs + write test plan

- **Goal**: take EXPLORE knowledge, decide PR boundaries + verification approach.
- **Output**: PR list (each <400 lines, single concern, independently shippable) + checkbox test plan per PR.
- **Exit**: slicing locked, test plan written.
- **Forbidden**: writing impl, writing tests, expanding scope. EXPLORE's job, not PLAN's.
- **Skipped signal**: PR > 600 lines, test plan empty at PR time, slicing decisions made mid-BUILD.

## 🔴 TEST — failing tests first

- **Goal**: write tests for next PR scope. Run → confirm fail. For non-code work (docs, config): lock contract + skeleton instead.
- **Output**: tests fail for the right reason (or contract + outline locked, body empty).
- **Exit**: red state confirmed — failure reason matches expected behavior.
- **Forbidden**: writing impl before tests fail. Pass-without-impl = test proves nothing.
- **Skipped signal**: tests written after impl, "I'll add tests later," coverage matches impl exactly (tests follow code instead of leading it).

## 🟢 BUILD — minimal impl to pass

- **Goal**: smallest code that turns red → green. Nothing more.
- **Output**: tests pass, no regressions, scope matches PLAN.
- **Exit**: full suite green for affected area.
- **Forbidden**: scope creep, features not in PLAN, refactoring unrelated code, premature optimization.
- **Skipped signal**: stuck in TEST forever, tests never go green, impl gets rewritten multiple times.

**Refactor sub-rules** (when BUILD is structural change, not new behavior):
- **Sprout**: new code path alongside old. Old untouched.
- **Migrate**: switch consumers one at a time. Each is its own PR.
- **Cleanup** of old path → CLEANUP phase, often becomes its own wheel rotation (separate PR).

## 🧹 CLEANUP — remove cruft

- **Goal**: delete old paths, scaffolding, debug code, dead code, commented-out experiments, TODO scaffolding.
- **Output**: codebase free of "needed during BUILD, not for shipping."
- **Exit**: nothing left to delete.
- **Forbidden**: adding features, changing behavior. Pure removal.
- **Skipped signal**: PR full of TODOs, commented-out code, debug logs, leftover scaffolding.

## ✅ VERIFY — confirm change works

- **Goal**: auto suite + manual checks against PLAN's test plan checkboxes.
- **Output**: green suite + all PLAN checkboxes ticked.
- **Exit**: ship-ready. PR description's verification list complete.
- **Forbidden**: skipping manual checks because "tests pass." Auto tests miss UX, stale reads, adjacent regressions.
- **Time-box manual**: per-PR budget (e.g. 5 min). Satisfaction-box ("test until I feel good") = where laziness wins.
- **Skipped signal**: bugs ship, follow-up tickets for "stale X" or "missed Y," prod incidents, "I forgot to test that."
