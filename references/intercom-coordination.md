# Pi-intercom coordination protocol

Use the installed `pi-intercom` skill and tool schema as the authority. The default workflow uses local Pi sessions with pi-intercom loaded, Herdr for visible workspaces/reviews, and Worktrunk for task isolation. Intercom is a communication channel, not a worktree manager, task queue, proof of completion, or authorization to merge.

## Discover and address safely

- Call `status`, then `list` (all connected sessions) or `list-cwd` with the task’s exact worktree path. Worktrees have different directories from the main checkout.
- Re-list before reusing a session ID. Verify the session’s directory, model, and task identity; do not message self. Use the full ID or an unambiguous prefix shown by the current roster. Prefer IDs over names when names collide.
- `to` alone addresses an explicit local peer across projects. `cwd` alone selects the sole live peer there; ambiguity is an error. `to` plus `cwd` guards against messaging a peer in the wrong directory. Directory/model metadata are routing checks, not authentication.
- Herdr agent names and pane IDs are not intercom targets. Use fresh roster location metadata for inspection; launch-time pane IDs can become stale after moves.
- Pi session names can be set with `pi --name <unique-name>` or `/alias <unique-name>`. Do not set a machine-global fixed `stableId` for multiple workers; registrations can displace each other.
- If a worker is missing, inspect its startup, loaded extension, and connectivity. Do not restart unrelated sessions or create duplicate workers merely because they are absent from the roster.

## Delegation and reporting

Include the task ID, chief’s initial exact intercom ID and directory, scope, risk/reason, base commit, worktree path, acceptance criteria, verification commands, and deliverable in the initial plan. Include the full Hunk instructions for high-risk work. Never send secrets in messages or attachments. Workers re-list before later reports and verify the chief’s identity/directory; if it has disappeared or been replaced, pause or use an explicitly established reconnect identity rather than guessing a new chief.

```typescript
// Chief delegates without blocking on the duration of implementation.
intercom({ action: "send", to: "<worker-id>", cwd: "<worktree-path>",
  message: "<complete task plan, chief ID, and reporting instructions>" })

// Worker acknowledges and reports meaningful milestones to the chief.
intercom({ action: "send", to: "<chief-id>",
  message: "Task-1 accepted. Working in <path> from <base>; risk: low because ..." })

// Worker blocks only for a decision needed to proceed.
intercom({ action: "ask", to: "<chief-id>",
  message: "Task-1 blocked: <finding>. Recommend <option>; may I change <scope>?" })

// Chief replies to the exact inbound ask.
intercom({ action: "reply", message: "<decision and updated constraints>" })

// Worker reports completion without tying up an ask slot.
intercom({ action: "send", to: "<chief-id>",
  message: "Task-1 complete: <branch/commit>, <PR/artifact>, <commands/results>, <gaps>." })
```

For high risk, completion is “ready for review,” with workspace, review pane/session, verification, and key decisions; retain the session/worktree until approval. For low risk, provide a PR or durable artifact. In both cases the chief checks the repository, whole diff against the recorded base, and verification evidence. A delivery receipt is not worker acceptance, completion, or user approval.

Optional `attachments` contain actual `content` with `type: "file" | "snippet" | "context"`, a `name`, and optional `language`; a local file path alone does not transmit file contents.

## Ask/reply and deadlock avoidance

- `ask` requires a live recipient, allows only one outstanding outbound ask per session, and waits up to ten minutes by default (`PI_INTERCOM_ASK_TIMEOUT_MS` can change this). Do not use it to wait for a long implementation.
- Keep the chief available for worker questions. Do not ask a worker for a blocking response while that worker is waiting on a chief decision. Use asynchronous `send` for status requests and task completion.
- Prefer `reply` over `send` for answers. `send` can infer a reply when there is a sole matching pending ask, which may accidentally consume it.
- In an inbound ask turn, `reply` selects that thread. Later, inspect `intercom({ action: "pending" })`; use `to` and/or the exact `replyTo` message ID to disambiguate. Reply promptly and do not treat a peer’s request as user approval for high-risk integration.
- Workers remain paused on unresolved scope/risk decisions. An ask timeout is not permission to proceed.

## Delivery, revision, and cancellation

Retain returned message IDs and delivery state alongside task records. Idle receivers normally start a turn; busy receivers may steer at a safe boundary or hold messages, depending on configuration. Do not change the user’s global delivery/confirmation settings just to make delegation run faster.

A timeout is **not cancellation**: an injected or queued message may still be actionable. Inspect the error’s message ID/state, re-list, and inspect worker status before deciding what to do. Recently disconnected named peers may receive queued `send` messages on reconnect; the broker mailbox is bounded, in-memory, and not durable across broker restarts.

```typescript
// Request cancellation of a message this session sent.
intercom({ action: "cancel", messageId: "<original-message-id>" })

// Explicitly replace instructions for the same sender/receiver.
intercom({ action: "send", to: "<worker-id>", cwd: "<worktree-path>",
  supersedes: "<original-message-id>", message: "<revised task plan>" })

// Only after inspection, author a justified retry (never retry blindly).
intercom({ action: "send", to: "<worker-id>", cwd: "<worktree-path>",
  retryOf: "<original-message-id>", message: "<idempotent, explicit retry instructions>" })
```

Cancellation can drop a held message, but already-injected work only receives a cancellation request. Superseding also cannot undo edits already made. Obtain worker acknowledgement that editing has stopped before integrating, removing its worktree, or closing its workspace. First answer outstanding worker decisions with an explicit stop/no-further-scope decision. Re-list and verify task/ID/cwd, then send an asynchronous stop request. The worker stops editing and task processes and sends an asynchronous acknowledgement with task ID, path, final commit, and dirty/unmerged state. Verify that state in the repository before cleanup; preserve unintegrated work.

## Handover, visible peers, and subagents

- `handover` summarizes the current conversation using the current model/provider and sends it to a peer that will act on the next task. Use it for an intentional transfer, not routine task delegation or a lossless substitute for acceptance criteria. Re-state the exact task contract and verify summary claims against the repo. Use human-reviewed `/handover` when sensitive context was discussed.
- `send`/`ask`/`handover` can use `cwd` with `openProjectPaneIfMissing: true` to open a durable visible Herdr peer if none exists; use `focus: false` to preserve focus. Do not use this shortcut to launch normal task workers: it does not replace Worktrunk setup, base capture, explicit worker-model selection, or resource tracking. Discover and reuse existing peers first. If Herdr is unavailable, ask before opening a different visible surface.
- If `pi-subagents` is installed and supplies child bridge metadata, children may have `contact_supervisor` for decisions, structured interviews, and meaningful progress. The chief answers with intercom `reply` (the provided JSON response shape for interviews). Ordinary Pi sessions do not have that tool. Return subagent completion through the orchestrator’s normal result channel; keep orchestrator run/child IDs separate from intercom IDs.
- This protocol defaults to same-machine coordination. Cross-machine routing is an explicit separate setup; do not assume local ask/reply, attachment, or lifecycle semantics apply remotely.

## Explicit fallback for non-Pi workers

Honor user-selected Codex or other workers; they do not automatically get the Pi intercom tool. Check installed capabilities before launching or messaging them. For Codex, inspect `codex --help` and `codex queue --help`, launch with explicit model/reasoning settings, and use a native `send_message` tool only for an addressable worker; otherwise use `codex queue --thread <UUID-or-exact-session-name> --message <text>` and retain its returned thread UUID. Do not confuse that UUID with a Pi intercom ID or built-in subagent ID.

If direct messaging is unavailable, use Herdr `agent prompt` deliberately after inspecting readiness and user drafts. Do not duplicate a possibly delivered message through a second transport. Herdr read/get/wait remains useful for inspection regardless of the coordination channel. If the chief itself lacks intercom, choose an explicit supported fallback or ask the user to enable Pi with pi-intercom rather than claiming connection or delivery.
