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

## Rollback Protocol

Test needs change during impl (wrong assertion/setup/misunderstood behavior):

1. **Revert impl** — undo all source changes, pre-fix state.
2. **Fix test** — correct expected behavior.
3. **Run test** — MUST FAIL against unfixed code. Critical. Passes without fix = test proves nothing.
4. **Re-implement** — apply fix again.
5. **Run test** — confirm passes.

Never skip "verify fail without fix" step.

## Test Quality

- One assertion per test. Clear name = expected behavior.
- Deterministic. No timing/random/network without mocks.
- Match project test framework, file structure, naming.
- Cover: happy path, empty/null/zero, boundaries, errors, all branches.
- Minimal setup. Don't over/under-mock.
- Test behavior not implementation. Refactors shouldn't break tests.

## Regression

After any fix/change: run FULL test suite for affected area. No regressions.
