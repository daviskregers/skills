---
name: tdd
description: >
  TDD rules: red-green-refactor, test rollback protocol, test quality.
  Use when writing tests, fixing bugs, or test-alongside-implementation workflows.
---

TDD rules. Apply to all test-writing workflows.

## Red-Green-Refactor

1. **Red** — write test that FAILS. Run. Confirm failure.
2. **Green** — minimal impl to pass. Run. Confirm pass.
3. **Refactor** — clean up. Run. Confirm still pass.

Test MUST fail before fix, pass after. Both states demonstrated. No fail→pass transition = test validates nothing.

RED must exercise the REAL production code, not a duplicate of the logic copied into the test file — a test that passes immediately proves nothing. If the real code is hard to bootstrap (facades, framework glue, UI/keymap/"shell" wiring), that's the signal to extract the logic into a real collaborator (a unit with its own reason to exist) and test THAT — not to skip. No layer is exempt: "it's just wiring/UI" is not an exemption. Make the thin remaining shell fail loud (visible error/notify), never swallow.

## Rollback Protocol

Test needs change during impl (wrong assertion/setup/misunderstood behavior):

1. **Revert impl** — undo all source changes, pre-fix state.
2. **Fix test** — correct expected behavior.
3. **Run test** — MUST FAIL against unfixed code. Critical. Passes without fix = test proves nothing.
4. **Re-implement** — apply fix again.
5. **Run test** — confirm passes.

Never skip "verify fail without fix" step.

## Test Quality

- One behavior per test (multiple assertions on that one behavior are fine — e.g. status AND error field; don't bundle unrelated behaviors). Clear name = expected behavior.
- Deterministic. No timing/random/network without mocks.
- Match project test framework, file structure, naming.
- Cover: happy path, empty/null/zero, boundaries, errors, all branches.
- Minimal setup. Don't over/under-mock.
- Test behavior not implementation. Refactors shouldn't break tests.
- No production test seams — never add wrappers, protected override hooks, or DI whose sole purpose is testability; nothing in production should reference "tests" or carry test-only indirection. Extract a real collaborator only if it earns its keep; else leave a small, shallow branch untested rather than deform the source.
- No tautological tests — don't assert schema/structure a migration just created (`hasColumn` after adding it); test the behavior that depends on it. Migrations themselves: write directly, no preceding test; cover downstream behavior (relations, roundtrips, controller writes).
- No branching logic in a test — no `if`/loops/conditional expectations computed in the test body. Behavior is fixed and known; cover branches as separate cases, not a conditional inside one test.

## Contract Migration

Existing behavior has tests but contract must change:

1. **New tests first** — write tests matching new contract expectations. Run → confirm FAILS.
2. **Implement** — change code to satisfy new contract. Run → new tests pass.
3. **Audit old tests** — review broken old-contract tests one by one:
   - Failure makes sense (contract changed)? Remove test or rewrite to mirror intent under new contract.
   - Failure unexpected (unrelated breakage)? Fix — regression found.
4. **Full suite green** — all tests pass, no old-contract tests left unreviewed.

Key: old test removal only after verifying new contract tests cover same behavioral intent. Don't lose coverage silently.

## Regression

After any fix/change: run FULL test suite for affected area. No regressions.
