# Deciding What to Test

Read this before adding tests or starting a TDD cycle. Choose useful proof first; test-first is a workflow, not a reason to create a test.

## Decision Gate

Before adding a test, answer:

1. What observable outcome or independently meaningful constraint should remain true?
2. What plausible accidental failure would this test detect?
3. Why would existing coverage not already detect that failure?
4. Would a validator, typecheck, linter, command execution, or dry-run provide more direct proof?
5. Does this protect a current requirement, or merely freeze an implementation choice or preference?

If the test adds no distinct protection, do not add it. Extend existing coverage when that is the clearest owner of the behavior. There are no quotas per function, method, or changed file.

## Choose the Proof

| Change | Starting point |
|---|---|
| New observable behavior | Focused behavioral tests; use test-first when practical |
| Bug fix | Reproduce the actual failure with a focused test when practical, then verify the fix |
| Behavior-preserving refactor | Existing coverage; add tests only for meaningful uncovered risks |
| CI workflow wiring | Workflow validation and execution of relevant commands |
| Lint configuration | Run the actual linter against affected code; do not retest upstream rules |
| Custom lint rules or tooling logic | Behavioral cases exercising your accepted/rejected inputs, outputs, exit codes, or side effects |
| Release automation | Validate configuration; test meaningful branching, permissions-related constraints, recovery, or cleanup where locally exercisable |
| Human documentation | Review accuracy and relevant examples; no source-text tests merely to preserve wording |
| Generated artifacts | Validate generation and meaningful consumer contracts, not copied inventories |

Configuration and CI can enforce important security, release, or compatibility contracts. Their file type does not exempt them from testing; equally, changing them does not automatically require new tests. Prefer the most direct independent proof. A narrow characterization test is appropriate when a non-obvious dependency assumption matters and is not already covered.

Choosing validation instead of a new test does not require permission merely to skip TDD. Respect repository requirements and separate approval requirements for external, destructive, or operational actions. Report what was checked and what remains unverified.

## Accidental Breakage Versus Intentional Change

A credible regression is a plausible accidental loss of previously working behavior that is still required. A new feature needs tests for its new contract, not an invented regression story. A changed preference or architecture decision is not itself a regression.

Test the outcome, not the historical choice:

- Replacing a singleton with dependency injection does not justify a test forbidding singletons.
- Switching providers does not justify asserting that the previous provider's name never appears.
- Replacing storage may justify testing that sessions still persist, not asserting the new adapter's private structure.

Independent architecture constraints can deserve checks when they are genuine current requirements, such as preventing client bundles from importing server secrets. Explain the constraint and failure mechanism rather than preserving an arbitrary arrangement.

## When Requirements or Direction Change

Reassess existing tests against the current intended contract:

- **Keep** tests protecting outcomes that still matter.
- **Rewrite or replace** tests whose expected behavior legitimately changed.
- **Delete** tests covering removed behavior or obsolete implementation choices.
- **Add** tests for new contracts when they provide distinct, meaningful protection.

A failing old test is not automatically evidence that the new implementation is wrong. First determine whether its requirement still applies. Do not preserve yesterday's preference as permanent law.

The justification for changing or removing a test must be the changed requirement, not merely making the suite green. Do not weaken coverage to conceal accidental breakage. Preserve coverage for retained and replacement behavior. Follow applicable confirmation requirements before deleting tests or files.

## Completion

State which contract or risk was checked, the exact checks performed, their results, and any meaningful gaps. More tests and higher test counts are not completion criteria. Once a test earns its place, follow the behavioral assertion and mock guidance in `writing-good-tests.md`.
