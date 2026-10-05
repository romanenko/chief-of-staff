# Pi Babysit coordination protocol

Use the installed `@yusukeshib/pi-babysit` README and loaded tool schema as the authority. This protocol was checked against pi-babysit 0.6.18 and babysit 0.14.6. The extension requires babysit 0.13.0 or newer; it does not install the binary for you. Do not upgrade packages or change global settings merely to work around a missing capability.

## Two session kinds, one supervised substrate

| Kind | Launch | Completion | Coordination |
| --- | --- | --- | --- |
| Process | `babysit_run { command }` | Command exits, or an explicit readiness wait matches | Exit notification; PTY text/keys for input |
| Pi subagent | `babysit_run { profile: "subagent", task }` | Current RPC task truly settles | Collect with `babysit_wait`; steer or give a follow-up with `babysit_send` |

Both have stable returned IDs, captured logs, state inspection, time bounds, and verified termination. Process supervision also covers builds, tests, servers, installers, REPLs, Git/GitHub commands, and non-Pi agents. It is not permission to merge, a filesystem sandbox, or a substitute for checking a result.

Babysit does **not** discover arbitrary peer Pi sessions or provide worker-to-worker/worker-to-chief send, ask/reply threads, mailboxes, or automatic conversation handover. The chief owns its children, reads their output, collects their results, answers decisions, and relays dependencies. A child can manage its own shell processes, but do not assume it can address the chief's namespace through its tools. Ordinary progress text is observable output, not an unsolicited message that necessarily starts a chief turn.

## Identity, recovery, and the task ledger

- Give meaningful runs names such as `task-1-implement`, `task-1-tests`, `pr-123-ci-r1`; always retain the **returned ID**. Duplicate names may be uniquified. Pi session IDs, Babysit run IDs, Herdr pane/workspace IDs, and PR numbers are distinct.
- `babysit_check { state: "all" }` inventories the current Pi-session namespace, not all agents on the machine. Before reuse, inspect the exact ID and check its task/revision, worktree instructions, model, and current task state. A supervisor that is still `running` can host an idle completed agent.
- Logs/state live under `~/.pi-babysit/<pi-session-id>/` by default (`PI_BABYSIT_DIR` can change the root). Retain the returned absolute log path. The raw CLI defaults to a different root, `~/.babysit`; do not use its default inventory to infer that Pi workers are missing.
- Same-session reload/resume reconnects to the namespace. `/new` or a fork does not make the old IDs part of the new inventory; retain the original root/session identity and inspect before launching replacements. A real Pi quit terminates its running processes and subagents. This is not an always-on service surviving coordinator shutdown.
- Keep a durable ledger outside task diffs: task/revision, constraints/risk, worktree/base/branch, parent namespace, run IDs/kinds/models, owned services, optional UI resource ownership, collected result, PR/head, check/review/approval evidence, and next event/action. Logs are automatically retained only temporarily; copy important evidence into a durable report.

## Launch and isolate workers

A reusable worker uses Pi RPC and is visible in Pi's Babysit widget. It does not need `herdr agent start` or a dedicated terminal workspace.

```typescript
// Replace the illustrative limits to fit the task; do not paste placeholders.
babysit_run({
  name: "task-1-implement",
  profile: "subagent",
  model: "openai/gpt-6.1-sol:medium",
  task: "<complete contract, absolute worktree path, reporting/stop rules>",
  timeout: "45m",
  reapAfter: "10m",
  maxTurns: 40
})
```

The loaded tool has **no `cwd` or `thinking` field**. Both process and subagent launches start at the caller's cwd. Use a Pi model's supported `:medium` suffix for subagent reasoning. A named `agent` definition is optional: the current implementation reads `name`, `description`, `model`, `tools`, and the Markdown system-prompt body, not a cwd override. Omit `agent` unless you verified the definition exists; `agentScope` defaults to `user` (`~/.pi/agent/agents`), with explicit `project`/`both` scopes for project definitions.

For a reusable child assigned to a different worktree, the contract must require:

1. Read the assigned worktree's `AGENTS.md`/other applicable instructions and verify its branch and recorded base using `git -C <absolute-path>` before editing.
2. Use absolute paths under that worktree for every read/edit/write. Use `cd <absolute-path> && <command>` or an explicit command cwd flag for **every** shell operation. Never use the chief's checkout for edits, checks, commits, or PR commands.
3. Recognize that a shell `cd` lasts only for that command. It neither changes future tool paths nor reloads worktree-local Pi settings, extensions, skills, or trust context. The task path is a contractual boundary, not an enforced sandbox.

If startup cwd/project discovery matters, use process mode instead:

```typescript
// <prompt-file> is a complete task contract stored outside reviewed changes.
// Quote real paths correctly. Record the Pi session path/ID for explicit resume.
babysit_run({
  name: "task-1-project-worker",
  command: "cd '<worktree-path>' && pi --mode json --model openai/gpt-6.1-sol --thinking medium --session-dir '<task-session-dir>' --name task-1 @'<prompt-file>'",
  timeout: "45m"
})
```

This is a one-shot process: collect process exit and the final JSON-mode assistant result, with bounded output or a durable artifact. A process exit code is not proof of task success. It has **no** subagent `mode: "task"` reuse semantics or automatic nested-usage accounting. Follow-ups launch another supervised command using the recorded exact Pi session (`--session <path-or-id>`), cwd, model, and revised contract; do not use an ambiguous `--continue`. Never type ordinary task text into a JSON/RPC protocol stream. Inspect `pi --help` and project trust before launch; do not bypass trust prompts automatically.

## Assignment and worker reporting contract

Include the goal, ordered plan, allowed files/actions, task ID/revision, absolute worktree, base commit, model, risk/reason, repository instructions, decisions, acceptance criteria, exact verification commands, and required artifact. State whether creating a commit, pushing, opening a PR, modifying a PR, or merging is authorized. Never put secrets in prompts or logs. Copy the full high-risk Hunk handoff with concrete review-host IDs when applicable.

Copy these reporting rules into each worker's contract:

> Acknowledge task ID/revision, path, branch/base, and risk in your output. Report meaningful milestones briefly. Stay within scope; do not spawn other agents, integrate, or remove resources unless explicitly delegated. If a required decision, capability, or higher risk blocks you, stop editing, report `BLOCKED` with evidence, options/recommendation, and the exact decision needed, and finish the current task. Do not spin or wait for an ask/reply channel. On an authorized PR opening, report `PR_OPEN` with repository, number/URL, head SHA, branch/commit, verification and gaps, and finish that phase immediately so the chief can start monitoring. For high risk, return `READY_FOR_REVIEW` with Hunk workspace/pane/session and decisions. Otherwise return `COMPLETE` with the durable artifact and evidence. Leave integration and cleanup to the chief.

For dependencies, collect the producing phase first, verify its commit/artifact, then relay the specific result in the dependent worker's task. Avoid two writers in one worktree. Escalate new scope/risk rather than allowing peer approval to replace a chief/user decision.

## Communication and event-driven scheduling

```typescript
// Correct constraints during the active task; not an immediate abort/rollback.
babysit_send({ id: "<returned-worker-id>", mode: "steer",
  text: "Task-1 revision 2: <updated constraints>. Acknowledge; stop if blocked." })

// Collect the next phase/result. Handle a blocker before waiting on the rest.
babysit_wait({ ids: ["<worker-1-id>", "<worker-2-id>"], mode: "any" })

// AFTER collecting this worker's prior task and verifying settlement:
babysit_send({ id: "<settled-worker-id>", mode: "task",
  text: "Task-1 revision 3: <decision/fix contract, evidence, exact checks>" })
// Every new background task needs collection too.
babysit_wait({ id: "<settled-worker-id>" })
```

Use explicit `steer`/`task` to express intent. `auto` steers unless confirmed settled; `task` rejects busy, parked, or unknown state. Steering is queued between turns, not guaranteed immediate compliance. Collect a previous task before starting the next: follow-ups change the task result window. If the child self-reaped, first inspect state/worktree and preserve its result, then launch a replacement with a reconstructed contract; do not pretend to resume the RPC worker.

Background subagents emit a ready-to-collect reminder, but **must still be collected** with `babysit_wait` before the parent finishes. Wait on at most 32 unique IDs per batch. `mode: "any"` does not cancel the others; keep them on the ledger and collect each. Prefer `all` only for independent bounded checks with no unresolved decisions. A blocked worker should finish with `BLOCKED`, enabling collection and a decision rather than deadlocking the chief.

A child may park its turn on its own background shell process. The extension recognizes this and waits for the child to resume on process exit; apparent RPC settlement in that parked turn is **not** task completion. Do not send a new task or kill an apparently quiet parked child just because a build is slow.

For processes:

```typescript
// Start parallel checks only when you will do concrete non-polling work.
babysit_run({ name: "task-1-tests", command: "cd '<path>' && <test-command>",
  continueAfterStart: true, timeout: "20m" })
babysit_run({ name: "task-1-lint", command: "cd '<path>' && <lint-command>",
  continueAfterStart: true, timeout: "10m" })
babysit_wait({ ids: ["<tests-id>", "<lint-id>"], mode: "all" })

// Service readiness is distinct from service exit.
babysit_run({ name: "task-1-dev", command: "cd '<path>' && <dev-command>",
  continueAfterStart: true })
babysit_wait({ id: "<dev-id>", expect: "<specific-ready-regex>", timeout: "2m" })
```

Ordinary background process starts end the chief's turn; exit automatically resumes it. Do not poll `babysit_check` or run a sleep command just to wait. `continueAfterStart` is for launching siblings, collecting results, or doing specific independent work; not a waiting loop. Readiness `expect` returns on a regex and leaves the process running; retain ownership and stop it later. Confirm application readiness rather than matching echoed input.

Related finite processes can share `notificationGroup` to notify once every running group member has stopped; sibling background calls are auto-grouped when omitted. **Do not group a PR observer with a long-lived server, repository overview, or slower independent PR**: it would delay the actionable notification. Assign separate explicit groups when early independent events matter.

## Bounds, inspection, and failures

- Give bounded tasks at least one realistic cost/turn/tool-call/token budget. Budgets are observed, steer toward wrap-up at 80%, and can overshoot during an in-flight call/batch. `maxUsageTokens` counts cumulative input, output, and cache usage across calls, not just generated tokens or the context window; a small token cap can kill a worker after very few turns. Size it above startup context and expected calls, or prefer a sensible turn/tool/cost cap. Inspect partial work on budget termination; never mark it successful.
- Subagent `timeout` defaults to 15m; process timeout defaults to none. `reapAfter` defaults to 120s of **finished task idle**, not lack of output. `reapAfter: "none"` disables idle reaping but not the absolute timeout. Choose both for anticipated PR fixes/human review. Long waits belong in cheap supervised processes, not model reasoning loops.
- Leave `idleTimeout` unset for quiet builds, servers, and CI/review watchers. It kills after **no output**, even during legitimate work; use only for a command expected to stream steadily. Keep recursion at depth 1. Only the chief may opt in to a bounded tree with `maxDepth`; nested workers cannot raise that ceiling.
- Tool allowlists replace selection. If restricting tools, include the file tools needed plus appropriate `babysit_run/check/wait/send/kill` operations; omitting `read` also breaks oversized file-backed task delivery. Do not silently remove supervision from workers.
- Use `returnPattern`/`returnLines`/`maxBytes` on foreground **process** runs; use `babysit_check { id, pattern: "FAIL|ERROR|BLOCKED", lines: 30, maxBytes: 6000 }` for purposeful log inspection. Output is bounded and full logs stay on disk. Do not load entire large logs or RPC JSON tails. Use targeted checks in fix loops and one full validation suite at the end.
- A wait timeout or interrupted `babysit_wait` does **not** cancel its process. Inline process ownership defaults to `lifecycle: "attached"` and terminates the tree if the owning run call is interrupted. Background starts are detached; detached does not mean surviving a real Pi quit. Use `babysit_kill` when termination is intended and inspect terminal state.
- Unexpected supervisor loss is reported as `worker-dead`, not an indefinitely running job. `retryOnWorkerDeath: true` is a once-only process retry for known safe, idempotent commands. Never apply it blindly to commits, pushes, PR creation, merges, deploys, or destructive actions. On ambiguous delivery/startup errors, inspect before resubmitting through another transport.

## Visible terminals, Hunk, and non-Pi workers

Herdr is a presentation/review surface, not the default message channel or worker supervisor. Use it only with its skill and `HERDR_ENV=1`. Run control/inspection/wait commands through Babysit; a Herdr launch command returning success only confirms that command's result, not the launched agent's completion. Preserve drafts and user focus.

For a task needing a visible workspace or Hunk review, open its **Worktrunk-created** worktree:

```text
herdr worktree open --workspace <chief-workspace-id> --path <path> --label "<task>" --no-focus
```

Resolve the main workspace from caller context/live state, not UI focus. Parse `.result.workspace` and `.result.root_pane`. Verify in `herdr workspace list` and `herdr worktree list --workspace <chief-workspace-id>` that main/worker share `worktree.repo_key`, the worker is linked, and its path matches. If main metadata is missing, inspect `worktree open --help`, register/reuse the main checkout with `herdr worktree open --cwd <repo> --path <repo> --no-focus`, rediscover IDs, and verify grouping. Stop if the installed CLI/server cannot support it; do not silently create flat workspaces or upgrade/restart Herdr. Worktrunk still owns creation/removal.

On `already_open`, inspect and reuse rather than overwriting an occupied pane or claiming ownership. Give a headless worker an explicit **review-host pane** in this workspace; its inherited `HERDR_PANE_ID` may point to the chief, not a worker terminal. Do not infer a pane from a Babysit ID.

Humans can use `/babysit`, `/babysit view`, and the process attach command shown there. A separate Herdr pane can display an existing process's attach view without launching another worker. Pi RPC subagents are read-only in the normal viewer: do not type into their protocol stdin or drive them as ordinary interactive Pi TUIs. Use `babysit_send` for subagent steering.

Honor explicitly requested Codex/other workers by inspecting their CLI and starting the native non-interactive command with explicit model/settings inside `babysit_run { command }` at the correct cwd. Coordinate by collected output/artifacts and supervised explicit native resume invocations. They do not acquire Pi subagent messaging semantics. For a deliberately interactive **process**, inspect with `babysit_check { screen: true }` and use `babysit_send { text }`/`{ keys }` only after confirming readiness and absence of user drafts/approval dialogs. A TUI's idle screen is not process exit. Do not accept user approval dialogs on the user's behalf.

## Stop and hand back

Collect each child's last outcome. For a settled live worker, send a stop/handback task: stop editing, stop owned services, and report task/path, final commit, dirty/unmerged state, artifacts, and any remaining processes. Collect its acknowledgement, then kill the idle worker and task-owned watchers/services. For an unresponsive worker, explicitly terminate and verify state instead of waiting indefinitely for an acknowledgement; preserve and inspect partial work.

Verify nested process ownership too; do not assume a stop of one visible pane cleaned up every detached resource. Never clean up user-owned or reused resources. Integrate only with authorization, preserve unmerged work, remove the Worktrunk branch/worktree only after integration or approved abandonment, and close only task-created Herdr resources. Keep an open PR's worktree and evidence available for repairs. Neither a completion result nor CI grants merge approval.
