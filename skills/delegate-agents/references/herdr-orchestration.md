# Herdr Adapter

Use only when the user explicitly requests Herdr. Read the installed Herdr skill or runner documentation for its actual CLI, environment requirements, prompt delivery, and safety rules. If those instructions or the required environment are unavailable, explain the limitation; do not invent commands or silently switch runners.

Follow [worker-reviewer-workflow.md](worker-reviewer-workflow.md) for the full workflow and the parent skill for model selection and authority.

## Herdr-Specific Coordination

- Verify the required Herdr environment before launching agents.
- Create only needed panes or windows, record their IDs, and preserve user focus.
- Use the requested settings or configured runner defaults; verify actual configuration when available. This adapter does not prescribe a model or provider.
- Deliver prompts with safely literal-quoted text or a quoted heredoc when shell interpretation is involved. Backticks and command substitutions can corrupt prompts. Inspect received content when it contains code or Markdown.
- Confirm the agent received its task and began work; creating a pane alone is not dispatch.
- Inspect output before interpreting idle/working states. Use available notifications or bounded checks rather than long blocking waits.
- Use an explicit working directory for commands; a previous command's `cd` may not persist across tool calls.
- The launching coordinator owns pane cleanup. After collecting results, close task-created panes or windows no longer needed, including after failure. Never close preexisting or unrelated panes.
- Preserve worktree changes and branches. Removing temporary worktrees requires applicable approval and a safe preservation plan; never force removal to finish cleanup.

## Verification

Check task-created pane identities, prompt delivery, actual task start, requested configuration where exposed, collected results, and cleanup. Report retained panes and their explicit follow-up purpose.
