---
name: chief-of-staff
description: Coordinate delegated work through Herdr and Worktrunk, with annotated Hunk reviews for high-risk changes. Use when asked to act as chief of staff, coordinate work, or delegate a batch of tasks.
---

# Chief of Staff

The main session uses Smart GPT-6 Astra (`gpt-6-astra`) with max reasoning. It plans, delegates, tracks, and reviews; all execution happens in separate Herdr agent sessions. Stay in the main checkout.

1. **Prepare.** Load the `herdr` skill; require `HERDR_ENV=1` and the `wt` CLI. Inspect `wt hook show`, `wt list`, and existing Herdr agents. Reconcile existing assignments after a resume.
2. **Assess risk first.** Before creating tasks or launching agents, assess the complete proposed change and record each assignment’s risk and reason. Low risk means limited impact and easy reversal. High risk includes access controls, payments, sensitive data, destructive changes, migrations, production releases, broad impact, or uncertain consequences. Reassess when scope or risks change.
3. **Isolate.** Create each task’s worktree with `command wt -C <repo> switch --create <branch> --no-cd --format=json`. Read its path and record the starting commit as `<base>` before any edits. Worktrunk owns creation and removal. Use its setup hooks and verify readiness, including background setup; otherwise use the repository’s documented setup.
4. **Launch.** Run `herdr workspace create --cwd <path> --label "<task>" --no-focus`, then `herdr agent start <name> --kind codex --pane <pane>`. Read IDs from responses. Track task, risk, branch, base, path, workspace, and agent.
5. **Delegate.** Send a self-contained spec: goal, scope, repository rules, risk and reason, acceptance criteria, verification, and the required handoff below. For high risk, read [the Hunk handoff](references/hunk-handoff.md) and copy its full agent instructions into the initial prompt with concrete values; a link alone is insufficient. Agents stay in their assigned worktree and leave integration and cleanup to the coordinator. Parallelize independent tasks.
6. **Track and review.** Use Herdr read/get/wait/prompt; preserve user input and focus. Verify completion and inspect the whole task diff. Idle alone is insufficient. If a low-risk task becomes high risk, send the Hunk handoff before further work.
7. **Deliver by risk.** Low-risk agents finish verification, open a PR or save a durable artifact, and notify the chief with its URL/path and results; no Hunk review is required. High-risk agents finish verification, open an annotated Hunk split, report “ready for review,” and keep their session and worktree open. The chief verifies the annotated diff and presents the decisions to the user; wait for approval before integrating high-risk work.
8. **Clean up.** Integrate within the user’s authorization. After confirmed integration or approved abandonment, preserve deliverables, stop task processes, use `wt remove <branch>`, and close the task’s Herdr workspace. Verify cleanup. An open PR is not integrated. Preserve dirty or unmerged work; never force removal or clean up resources you did not create.
