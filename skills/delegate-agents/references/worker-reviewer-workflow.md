# Worker / Independent Reviewer Workflow

Use for an explicitly requested full implementation → review workflow. The parent skill governs model selection, authority, and runner choice.

## Procedure

1. Confirm objective, approved approach, boundaries, and required approvals. Inspect the canonical checkout, branch, and dirty state; follow repository branch policy without moving unrelated changes.
2. Establish coordinator ownership of architecture, shared interfaces, integration order, and consequential choices. Let workers own local implementation details. Escalate unresolved product or scope questions.
3. Start the worker with a decision-complete handoff and verify receipt and task start. Use isolated checkouts or explicit file ownership for concurrent work. Set a concrete first milestone for larger tasks.
4. Monitor meaningful outputs using the runner's notification or bounded status facilities. Answer approved choices directly. If planning stalls, ask for a bounded deliverable; do not impose new tests merely to force an artifact.
5. Collect the implementation and validation report, inspect tracked changes and untracked files, and freeze edits during review. Start a separate fresh read-only reviewer unless review was explicitly excluded. Provide the base revision, full change inventory, relevant contracts, runtime boundaries, and review method.
6. Ask for actionable findings with file/line evidence and a short failure mechanism or reproduction. Separate confirmed bugs from unverified risks. Reviewers advise the coordinator; they do not modify the checkout or instruct workers directly. Permit local checks only when safe within the approved constraints; read-only review does not authorize installs, formatting, generation, Git mutation, or publication.
7. Adjudicate findings against the actual request, diff, and current contracts. Timebox speculation. Send confirmed fixes to the worker, then independently review the final change again. Verify integrated behavior with relevant checks; if validation would mutate a frozen checkout, coordinate it separately. Do not blindly accept the reviewer's conclusions or quietly take over implementation.
8. Report changed files, decisions, exact validation and skipped checks, unresolved risks, Git state, and external effects. Recheck checkout status because concurrent work can invalidate earlier observations. Clean up only task-created resources no longer needed; retain resources only for explicit follow-up and preserve all work.

## Pitfalls

- An idle state can indicate a pending question; a working state does not prove progress.
- `git diff` omits untracked files. Include them in review scope.
- A sibling worktree's dirty state is not the current checkout's dirty state.
- Replaying an obsolete architecture preference as a regression test freezes yesterday's decision. Protect current outcomes instead.
- Configuration edits often need validators or command execution, not new unit tests.
- Cleanup must not discard changes or close unrelated user sessions.

## Verification

Confirm distinct worker/reviewer identities, delivered handoffs, final review coverage including untracked files, re-review of confirmed fixes, integration checks, final checkout status, and lifecycle cleanup. Report any unavailable checks rather than claiming completion from agent status alone.
