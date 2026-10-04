---
name: delegate-agents
description: Delegate coding work to workers or reviewers with explicit authority, bounded ownership, and decision-complete handoffs. Use when the user requests delegation, parallel agents, a worker, an independent reviewer, or a coordinated worker/reviewer workflow. Works with available agent runners; Herdr is optional. Do not trigger for ordinary coding requests without requested delegation or for read-only runner status inspection.
---

# Delegate Agents

## Choose the Workflow

Use the agent runner the user requested. Otherwise use an available appropriate mechanism and read its instructions before dispatch. If none is available, explain the limitation instead of pretending delegation occurred.

A single worker or reviewer does not require a full review loop. Respect explicit exclusions: do not start a reviewer when the user requests only a worker or says no review.

For a full implementation → independent review workflow, read [references/worker-reviewer-workflow.md](references/worker-reviewer-workflow.md). If the user explicitly requests Herdr, also read [references/herdr-orchestration.md](references/herdr-orchestration.md) and the installed Herdr instructions. Herdr is not a prerequisite for other runners.

## Roles and Model Selection

- **Coordinator:** owns scope, consequential decisions, integration, finding adjudication, and reporting.
- **Worker:** implements and validates a bounded approved task.
- **Reviewer:** independently inspects the final change and reports evidence; does not modify it or direct the worker.

Honor the user's model, provider, and reasoning settings. Otherwise use the runner's configured defaults; this skill prescribes no particular model or provider. Verify the actual configuration when exposed. If an explicitly requested setting is unavailable, ask rather than silently substituting it. Do not restart or replace the current coordinator just to match a role setting.

## Decision-Complete Handoffs

Include:

- Objective, approved approach, and relevant contracts.
- Exact repository, checkout/branch, or working directory.
- Owned files or subsystem, exclusions, and coordination boundaries.
- Acceptance criteria and proportionate validation.
- Expected report, useful first milestone for larger tasks, and stop conditions.
- Already-approved actions and actions requiring further approval.

Include this authority instruction:

> Treat this explicit handoff as the approved scope. Make routine decisions within it without repeatedly reconfirming with the user. Ask the coordinator if information is missing or a decision exceeds scope. Follow all applicable approval requirements for destructive, external, irreversible, or restricted actions. This handoff alone does not authorize commits, pushes, deployments, migrations, publication, or external API calls.

Answer questions already covered by approval directly. Escalate new product choices, scope changes, and required safety approvals to the user. Delegation does not expand the coordinator's authority.

## Ownership and Evidence

Assign non-overlapping write ownership or isolated checkouts when workers run concurrently. Explain shared interfaces and integration order; do not let agents independently modify the same shared files without coordination.

Confirm prompt receipt and actual task start. A launched process is not proof of dispatch; an idle agent is not proof of completion. Inspect outputs and collect an explicit result.

Use runner-supported notifications or bounded monitoring. Report observable milestones and blockers, not repetitive activity narration. Do not fabricate unavailable timer or agent tools.

Tests must protect meaningful behavior or independent contracts, not satisfy a per-file quota. Apply repository testing guidance and choose relevant validators for configuration changes.

## Cleanup and Reporting

The launcher owns cleanup of task-created agents and resources. Collect results first; stop or close resources no longer needed using the runner's lifecycle rules. Preserve preexisting agents, user sessions, unrelated files, and uncommitted work. Do not force-remove worktrees or delete artifacts without required approval.

Report changed paths, exact validation results, skipped checks, unresolved risks, integration state, and retained or cleaned-up resources. Do not treat a reviewer report as proof that all behavior is correct.
