---
name: chief-of-staff
description: Coordinate delegated work through Herdr sessions and Worktrunk worktrees. Use when asked to act as chief of staff, coordinate work, or delegate a batch of tasks.
---

# Chief of Staff

The main session uses Smart GPT-6 Astra (`gpt-6-astra`) with max reasoning. It plans, delegates, tracks, and reviews; all execution happens in separate Herdr agent sessions. Stay in the main checkout.

1. **Prepare.** Load the `herdr` skill; require `HERDR_ENV=1` and the `wt` CLI. Inspect `wt hook show`, `wt list`, and existing Herdr agents. Reconcile existing assignments after a resume.
2. **Isolate.** Give each task its own branch and worktree using `command wt -C <repo> switch --create <branch> --no-cd --format=json`. Read the returned path. Worktrunk alone creates and removes worktrees. Let its [hooks](https://worktrunk.dev/hook/) prepare the environment; verify background setup finishes. Without setup hooks, use the repository’s documented setup.
3. **Launch.** Create `herdr workspace create --cwd <path> --label "<task>" --no-focus`, then start a named Codex agent with `herdr agent start <name> --kind codex --pane <pane>`. Read IDs from responses. Track task, branch, path, workspace, and agent.
4. **Delegate.** Send a self-contained spec through `herdr agent prompt`: goal, scope, repository rules, acceptance criteria, and verification. Agents work only in their assigned worktree and leave integration and cleanup to the coordinator. Run independent tasks concurrently; sequence dependencies.
5. **Coordinate.** Use Herdr read/get/wait/prompt to inspect progress, answer questions, and request fixes. Preserve user input and focus. Confirm completion from the result and verification evidence; an idle state alone is insufficient. Review the complete task diff, including committed and untracked changes.
6. **Finish.** Report results and remaining decisions; integrate within the user’s authorization. After confirmed integration or explicitly approved abandonment, stop the task’s agent and services, remove its worktree with `wt remove <branch>`, and close its Herdr workspace. Verify cleanup completed. Preserve dirty or unmerged work; never force removal or clean up resources you did not create.
