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

For isolated work in this hosted workflow, require the [Herdr Worktrunk plugin](https://github.com/devashish2203/herdr-worktrunk) and repository setup hooks. Use its Worktrunk-backed actions for checkout creation/switching and authorized removal/merge; do not substitute Herdr's built-in worktree creation/removal. Herdr provides terminal layout; the plugin does not replace ownership, review requirements, or approval gates in `operations.md`.

## Hosted worker threads

1. Provision the assignment's working state and verify repository setup.
2. Create the requested topology with the correct cwd. For isolated worker worktrees, use the repository-grouped workspace procedure below rather than plain `workspace create`; read returned workspace and root-pane IDs.
3. Start the approved agent kind/model using installed syntax. For example, `herdr agent start <name> --kind <kind> --pane <pane> -- <native-agent-options>`. Do not guess flags or weaken approval/sandbox settings.
4. Confirm readiness, identity, and cwd before sending the complete spec through native messaging or a verified hosted adapter. Record acknowledgement and establish a separate recurring check-in task.
5. Keep disposable review artifacts under a git-excluded assignment location. Tell hosted workers how to read review comments, refresh their notes after changes, and report verification evidence.

Native messaging remains preferred. Hosted terminal delivery is not acknowledgement, and neither idle nor a settled wait is completion. Use terminal inspection only for supported adapters, liveness, human dialogs, and UI state.

Before delivery, inspect `herdr agent read <name> --source visible`. Protect actual pending human text; UI suggestions are not user input. On a blocked dialog, inspect recent unwrapped output and ask the user before answering. Rediscover identity after a lost connection; do not assume an error means the worker stopped.

## Required Worktrunk plugin and nested Spaces

Use [devashish2203/herdr-worktrunk](https://github.com/devashish2203/herdr-worktrunk) for the Herdr worktree workflow. Its default `open_mode = "workspace"` runs Worktrunk to create/switch the checkout and execute lifecycle hooks, then registers the checkout through `herdr worktree open`. Spaces displays nested worktree workspaces beneath the repository's main workspace. This is **repository/worktree grouping**, not an agent parent-child or messaging hierarchy; it applies to Pi, Claude, and Codex workers.

### Prerequisites and configuration

Read the installed plugin README and Herdr plugin CLI help; inspect `herdr plugin action list`. Upstream requires Herdr ≥ 0.7.0, Worktrunk ≥ 0.60.0 (`wt` on PATH), `fzf`, `jq`, and Bash on macOS/Linux. Verify these and the installed plugin before starting isolated hosted work. If missing, report the blocker and request authorization to install; do not silently bypass it or upgrade shared tools.

```bash
# Only with installation authorization:
herdr plugin install devashish2203/herdr-worktrunk
# Inspect the managed config location:
herdr plugin config-dir worktrunk
```

Require workspace presentation for this workflow. Inspect `config.toml` in the returned directory: absent `open_mode` defaults to `"workspace"`; if it is `"tab"`, request approval to change it to `"workspace"`. Do not silently edit global configuration or keybindings. Tab mode does not provide the required nested Spaces layout and additionally needs Worktrunk shell integration. Plugin config is read on each picker invocation; no reinstall/restart is needed. Inspect `merge_flags` before any merge: options such as `--no-hooks` or automatic staging must not bypass repository requirements or include unrelated work.

### Create/switch and verify

1. Resolve the Chief/main workspace from `herdr pane current --current` and live workspace state, never UI focus. Verify repository, source branch/base, and hook approvals before invoking an action in that repository context. Use installed targeting syntax; do not invent a branch or workspace argument.
2. Invoke the appropriate plugin action:

   ```bash
   herdr plugin action invoke open --plugin worktrunk
   herdr plugin action invoke open-current --plugin worktrunk
   herdr plugin action invoke open-with-remotes --plugin worktrunk
   ```

   `open` creates typed new names from Worktrunk's default base; `open-current` uses the current branch (`--base @`); `open-with-remotes` includes remote-tracking branches. Refresh remote state only when authorized/needed. These actions open an **interactive fzf picker**, not a noninteractive assignment API. Inspect the picker before input, preserve human drafts, and use a supported terminal adapter or ask the user to select/type the intended branch. `Enter` selects a match; `Alt+Enter` forces a typed new name when it fuzzy-matches an existing branch. Action-launch success is not checkout readiness.
3. Inspect Worktrunk/hook completion and discover the actual checkout path and workspace/root-pane IDs from live state; do not predict them. A failure keeps the picker pane open. `hold_on_create`/`hold_on_success` can retain successful hook output, but changing those settings requires authorization. Verify setup independently when output has disappeared.
4. Before launching the worker, inspect `herdr workspace list` and `herdr worktree list --workspace <chief-workspace-id>`. Main and worker must share `worktree.repo_key`; the worker must have `is_linked_worktree: true` and the assigned checkout path. Record branch/base, IDs, path, hook evidence, and resource ownership. Reuse an existing space only after inspecting its occupant; do not launch over an occupied pane or claim ownership of reused resources.
5. If main metadata is missing, inspect installed help and register/reuse the main checkout with `herdr worktree open --cwd <repo> --path <repo> --no-focus`, then rediscover IDs and verify grouping. Explicit registration of an existing checkout is a layout repair, not a replacement for the required plugin lifecycle. If grouping cannot be verified, stop and report the blocker; do not fall back to flat workspaces or restart Herdr.

Do not substitute plain `workspace create`, messaging auto-creation, built-in `herdr worktree create/remove`, or a hand-rolled `wt` lifecycle for the plugin workflow. Plain workspaces remain appropriate for explicitly requested unrelated topology. Worker communication and supervision remain independent of Spaces presentation. Preserve user focus where supported; picker actions may require interaction/focus, so do not promise they are background `--no-focus` operations.

### Authorized merge/removal and recovery

After shared acceptance, permission, dependency, and dirty/unmerged-work guards pass, use the plugin's installed actions:

```bash
herdr plugin action invoke remove --plugin worktrunk
herdr plugin action invoke merge --plugin worktrunk
herdr plugin action invoke merge-no-squash --plugin worktrunk
```

Inspect the manifest/action list before use. Merge actions run `wt merge` and removal; the no-squash variant preserves commits. They are **not** substitutes for PR protections, current-head checks, or merge authorization. Do not invoke merge merely to clean up an already merged PR. Stop the worker and owned processes before removal, verify the selected checkout, and never accept destructive prompts on the user's behalf without delegated authority. Worktrunk confirmation is not permission to discard unknown work.

The plugin closes associated native workspaces or legacy tab panes after successful removal; failed merge/removal retains work and UI. Verify hook results, checkout/branch disposition, and UI cleanup before declaring completion. Do not redundantly close a space already removed by the plugin. Never use `workspace close --group`: it can affect the Chief and siblings. Reused resources require explicit management authority.

On recovery, reconcile Worktrunk checkouts, grouped spaces, workers, and partially completed actions before invoking anything again. Do not duplicate an ambiguous create, merge, or removal; inspect first. Keep all general cleanup and approval guards in force.

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
