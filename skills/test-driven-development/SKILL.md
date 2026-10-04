---
name: test-driven-development
description: Use when implementing meaningful behavior changes or bug fixes. First decide whether new tests provide useful protection; then use focused test-first development where appropriate. Ordinary configuration edits and behavior-preserving refactors do not automatically require new tests.
---

# Test-Driven Development

## Choose Useful Proof First

Before adding tests or beginning TDD, read [deciding-what-to-test.md](deciding-what-to-test.md) and apply its decision gate. Test-first is a development workflow, not a justification for creating a test.

Protect observable behavior and independent contracts. Do not add tests simply because a function, file, workflow, or architectural choice changed. There are no per-function coverage quotas.

When writing or changing tests, also read [writing-good-tests.md](writing-good-tests.md). If the test-audit skill is available, apply its authoring gate as well.

## When to Use Test-First

Use focused test-first development for meaningful new behavior and reproducible bugs when practical:

- For a bug, reproduce the actual failure and confirm it fails for the intended reason before fixing it.
- For new behavior, express the desired contract independently of the implementation.
- For a behavior-preserving refactor, run existing coverage. Add tests only for meaningful uncovered risks.
- For ordinary CI, lint, and configuration changes, prefer appropriate validators and actual command execution. Custom tooling logic may still warrant behavioral tests.

Choosing appropriate validation instead of adding a low-value test does not require permission just to skip TDD. Follow repository policies and all separate confirmation requirements for edits, deletions, external calls, and operational actions.

If test-first is impractical, identify why and use the strongest feasible proof. Do not manufacture brittle tests to satisfy a process. Tests written after implementation still need independent expectations and evidence that they detect the intended failure.

## Red / Green / Refactor

### 1. Red: Express One Contract

Write the smallest useful test of an observable outcome or meaningful invariant. Name the failure it detects. Prefer real code and controlled inputs; mock slow or external boundaries only when needed.

Run the focused test. Confirm it fails because the behavior is absent or incorrect, not because of broken setup, a typo, or an unrelated error. A passing test may indicate existing coverage or an already implemented contract; investigate rather than forcing an artificial failure.

### 2. Green: Implement the Required Behavior

Write the simplest implementation satisfying the intended contract. Do not introduce unrelated features or refactors.

Run the focused test and relevant existing coverage. If a retained-contract test fails, determine the cause rather than weakening it. If requirements intentionally changed, follow the test-evolution guidance below.

### 3. Refactor Only Where Needed

Keep validated behavior unchanged. Refactoring is not mandatory, and this workflow does not authorize unrelated cleanup. Re-run relevant checks after changes.

## Existing Implementation and Exploration

Do not delete implementation merely because it preceded a test. Inspect the current behavior, assess its risks, and add justified proof. For a regression test, demonstrate that it catches the original failure using a safe baseline comparison or controlled mutation when feasible.

Exploration and prototypes can precede formal tests. Before shipping, validate the intended contracts with suitable tests or other checks. Do not rewrite working code solely to recreate a test-first sequence.

## Changed Requirements Are Not Regressions

An intentional architectural or preference change is not accidental breakage. Test persistent outcomes, not the absence of the previous design.

When direction changes, keep tests for retained contracts, rewrite or replace tests for changed behavior, delete obsolete coverage when authorized, and add useful tests for new contracts. See [When Requirements or Direction Change](deciding-what-to-test.md#when-requirements-or-direction-change).

Do not weaken tests merely to get a green suite. Equally, do not treat an obsolete test as an immutable requirement. Establish whether its contract still applies first.

## Completion Checklist

- The proof addresses current intended behavior or an independent constraint.
- New tests detect a distinct plausible failure not already covered.
- Expected results are independent of the implementation under test.
- Regression tests reproduce the actual failure where feasible; new behavior is not mislabeled as a regression.
- Relevant existing coverage and appropriate validators were run.
- Changed or removed tests are justified by changed requirements, not concealed bugs.
- Report exact checks, outcomes, and meaningful unverified risks.

Success means useful confidence with proportionate maintenance cost—not a test for every function or a ritual red/green sequence for every edit.
