---
name: tdd
description: >
  Test-driven development rules: red-green-refactor cycle, test rollback protocol,
  test quality guidelines. Use when writing tests, fixing bugs with TDD, or any
  workflow that involves writing tests before or alongside implementation.
---

Test-driven development rules. Apply to all test-writing workflows.

## Red-Green-Refactor Cycle

1. **Red** — write a test that FAILS. Run it. Confirm failure.
2. **Green** — write minimal implementation to make test pass. Run it. Confirm pass.
3. **Refactor** — clean up if needed. Run tests. Confirm still pass.

The test MUST fail before the fix and pass after. Both states must be demonstrated. If you can't show the fail→pass transition, the test is not validating anything.

## Test Rollback Protocol

If the test needs to change during implementation (wrong assertion, wrong setup, misunderstood behavior):

1. **Revert implementation** — undo all source changes, return to pre-fix state.
2. **Fix the test** — update so it correctly captures expected behavior.
3. **Run updated test** — confirm it FAILS against unfixed code. Critical step. If it passes without the fix, test proves nothing.
4. **Re-implement** — apply the fix again.
5. **Run test** — confirm it passes now.

Never skip the "verify test fails without fix" step. That's the whole point.

## Test Quality Rules

- **One assertion per test** — each test tests ONE thing with a clear name describing the expectation.
- **Deterministic** — no timing-dependent, no random values, no network calls without mocks.
- **Match project patterns** — use existing test framework, file structure, naming conventions in the project.
- **Happy path + edge cases** — cover: normal input, empty/null/zero, boundaries, error conditions, all branches.
- **Minimal setup** — test only what's needed. Don't over-mock, don't under-mock.
- **Test behavior, not implementation** — test what the code does, not how it does it. Internal refactors shouldn't break tests.

## Regression Check

After any fix or change, run the FULL test suite for the affected area — not just the new test. No regressions allowed.
