# Herdr-hosted coordination

## Scope and detection

Apply this reference only inside Herdr. Herdr is a terminal host, not an agent Harness; Pi, Claude Code, and Codex can each run inside or outside it.

```bash
test "${HERDR_ENV:-}" = 1
```

If the test fails, or environment access is unavailable, ignore all Herdr/Hunk workflows and use native Harness coordination and artifact review. Installed binaries, model names, and other live host sessions are not evidence that the Chief is hosted.

Before control actions, confirm the live caller with `herdr pane current --current`. If it cannot be resolved, stop host-specific actions. Do not control the focused session from an unhosted caller. Confirm workers are hosted before giving them terminal-specific instructions.

## Setup and ownership

Read the installed Herdr skill and current CLI help. Read the Hunk skill returned by `hunk skill path` before review-session operations. Verify supported commands rather than assuming a particular version.

Use explicit pane/session IDs or `--current`, parse returned JSON, preserve user focus with `--no-focus`, and record assignment ownership of created resources. Do not close unrelated panes/workspaces, restart a shared daemon, or terminate the host without explicit authorization.

For isolated work, use the approved worktree manager and repository setup hooks. Herdr provides terminal layout; it does not replace working-state ownership, review requirements, or approval gates in `operations.md`.

## Hosted worker threads

1. Provision the assignment's working state and verify repository setup.
2. Create the requested topology with the correct cwd. For isolated worker worktrees, use the repository-grouped workspace procedure below rather than plain `workspace create`; read returned workspace and root-pane IDs.
3. Start the approved agent kind/model using installed syntax. For example, `herdr agent start <name> --kind <kind> --pane <pane> -- <native-agent-options>`. Do not guess flags or weaken approval/sandbox settings.
4. Confirm readiness, identity, and cwd before sending the complete spec through native messaging or a verified hosted adapter. Record acknowledgement and establish a separate recurring check-in task.
5. Keep disposable review artifacts under a git-excluded assignment location. Tell hosted workers how to read review comments, refresh their notes after changes, and report verification evidence.

Native messaging remains preferred. Hosted terminal delivery is not acknowledgement, and neither idle nor a settled wait is completion. Use terminal inspection only for supported adapters, liveness, human dialogs, and UI state.

Before delivery, inspect `herdr agent read <name> --source visible`. Protect actual pending human text; UI suggestions are not user input. On a blocked dialog, inspect recent unwrapped output and ask the user before answering. Rediscover identity after a lost connection; do not assume an error means the worker stopped.

## Group worker workspaces in Spaces

For assignments using separate worktrees, register each existing worker checkout through `herdr worktree open`. This restores the proven workflow from commit `59fd184`: Spaces groups worker workspaces beneath the repository's main workspace instead of displaying unrelated flat spaces. This is **repository/worktree grouping**, not an agent parent-child relationship or a messaging hierarchy; it applies equally to Pi, Claude, and Codex workers.

1. Let the approved worktree manager create the checkout and run setup hooks first. Worktrunk, when selected, still owns creation/removal; do not use `herdr worktree create` as a substitute.
2. Resolve the Chief/main workspace from `herdr pane current --current` and the live workspace list, never UI focus. Inspect `herdr worktree open --help` and the installed worktree command group before applying this procedure.
3. Open the existing checkout under that repository context, preserving focus:

   ```bash
   herdr worktree open --workspace <chief-workspace-id> --path <worktree-path> --label "<task>" --no-focus
   ```

4. Parse `.result.workspace` and `.result.root_pane`. If the response reports `already_open`, inspect and reuse the existing workspace; do not launch over an occupied pane or claim ownership of reused resources.
5. Before starting the worker, verify with `herdr workspace list` and `herdr worktree list --workspace <chief-workspace-id>` that main and worker share `worktree.repo_key`, the worker has `is_linked_worktree: true`, and its checkout path matches the assigned worktree. Record returned IDs, path, repository key, and resource ownership in the assignment registry.
6. If the main workspace lacks repository metadata, register/reuse the main checkout using the installed syntax:

   ```bash
   herdr worktree open --cwd <repo> --path <repo> --no-focus
   ```

   Rediscover the main workspace/pane IDs and verify grouping again before launching workers. If the installed CLI/server cannot support registration, stop and report the blocker; do not silently fall back to flat workspaces, upgrade, or restart Herdr.

Do not use plain `workspace create` or messaging auto-creation for these worker workspaces: they can omit the repository metadata needed for grouping. Plain workspaces remain appropriate for explicitly requested unrelated topology. Worker identity, native communication, check-ins, and approval gates remain independent of Spaces presentation.

During recovery, reconcile existing grouped workspaces and live workers before opening replacements. During cleanup, close only the assignment-owned worker workspace after guarded worker/worktree disposal. **Never use `workspace close --group`** for task cleanup: it can affect the Chief/main workspace and sibling workers. A reused workspace is not a resource this assignment created.

## Hunk review and annotations

Hunk is optional review tooling within this hosted workflow. If unavailable, use an approved alternative that meets the same review/approval requirements; do not weaken those requirements.

For a Git working-state review:

1. Split beside the worker with `herdr pane split --pane <agent-pane> --direction right --cwd <path> --no-focus`, adapting geometry to the actual layout. Read the new pane ID.
2. Start `hunk diff HEAD` in that pane via `herdr pane run`. Including `HEAD` shows staged and unstaged changes. Discover the live review with `hunk session list` and target its exact ID if ambiguous.
3. Apply assignment notes with `hunk session comment apply --repo <path> --stdin`. Notes may be stored in git-excluded `.hunk-notes.json`:

```json
{"comments":[{"filePath":"src/example.ts","newLine":42,"summary":"What changed.","rationale":"Intent, risks, and what to verify.","author":"worker"}]}
```

4. Add Chief findings through the session CLI and report decisions required. Focus the review only when requested.
5. For revisions, have the worker read `hunk session comment list --repo <path> --type user`, make changes or reply in-thread, re-run verification, and report completion.
6. Refresh the live view from current files using the installed reload interface. Preserve annotations first; inspect all comments before replacing notes, avoid duplicate application, and never remove a note that parents a human reply. Do not clear human comments or restart the daemon without approval.

For committed work, review the appropriate base-to-branch diff. For files outside a Git repository, use a supported file/patch review instead of creating a repository in the user's skill directory. Keep snapshots current; a review snapshot is not automatically a live view of disk changes.

## Terminal PR and Actions watching

When `gh observer` is available, read [the separate gh-observer reference](gh-observer.md) for explicit PR/run targeting, PTY requirements, bounded watchers, repository overview, and current-head verification. Use a verified supervisor/wake-up path; opening a terminal viewer alone does not notify the Chief or monitor all comments/reviews. This optional CLI also works outside Herdr and does not replace native worker messaging or the shared PR-event contract.

## Harness adapters

- **Pi:** use intercom for conversation and managed processes for check-ins. Enable `openProjectPaneIfMissing` only within this verified hosting context and the assignment's topology/ownership plan.
- **Claude Code:** use exposed native messaging when it reaches the worker; match full worker identity and working path rather than truncated names. Deliver slash commands only through a supported terminal interface.
- **Codex:** use the installed agent adapter with approved native options. If native messaging cannot reach a separate hosted worker, use a verified prompt/read adapter and require structured responses.

## Recovery and disposal

After resume or host restart, reconcile caller location, agents, native addresses, panes, working state, and live review sessions. Wait/idle subscriptions may be lost and review panes may return to a shell. Recreate missing monitors without duplication; relaunch review UI only when wanted.

After confirmed merge or explicit rejection, stop assignment-owned monitors and services. Quit the assignment's Hunk session with `herdr pane send-keys <review-pane> q` and verify closure. Exit the worker only when safe with no pending human input, using that Harness's documented mechanism; do not assume one Harness's slash commands work in another. Verify termination before removing its cwd.

Apply shared cleanup guards, then close only assignment-owned workspaces with `herdr workspace close <workspace>`. Never use `workspace close --group` or close the Chief/main workspace, repository group, sibling workers, or a reused workspace. Preserve evidence and report any resource that cannot safely be removed. Never kill the Chief, the host, or unrelated sessions as cleanup.
