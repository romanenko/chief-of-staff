# Codex coordination and communication

Use this reference when the Chief runs in the Codex harness, not merely when another harness uses an OpenAI/Codex model. Read `coordination.md` for the shared lifecycle contract.

## Discover capabilities before launch

Inspect the installed native Codex tools and documentation/help. Use only supported agent launch, messaging, resume, background execution, and shutdown interfaces. Do not assume Claude's `ListAgents`/`SendMessage`, Pi's `intercom`/`process`, or a particular Codex multi-agent API exists.

Launch each implementation worker in isolated working state using the verified native Codex adapter/CLI. Select the user-approved model and preserve configured approval/sandbox rules; never substitute bypass flags for a missing capability. Confirm worker identity, cwd, branch, and readiness before delegation.

## Communication adapter

Prefer available native Codex worker messaging. Record the returned worker/session handle and use the installed API for specs, progress requests, review rounds, and ship prompts. Native child-agent tools may not address independent CLI/desktop sessions; verify that distinction.

If native messaging cannot reach an independent worker, surface the gap and use only an approved, verified bridge. Send a complete spec or a pointer to git-excluded `<worktree>/.review/SPEC.md`. Require structured replies containing task ID, milestone, evidence, blocker, and next action, delivered through the verified native channel/bridge or a status file observed by a verified monitor.

Message delivery is not acknowledgement. Track explicit responses and deadlines. Do not overwrite pending user text, inject prompts into approval dialogs, or answer a permission denial yourself. Do not assume an idle indicator or process exit means the task was accepted, verified, or merged.

For mixed-runtime workers, read their runtime reference too and verify a bridge/adapter. Pi intercom cannot address a Codex session just because it is on the same machine. Do not invent direct peer messaging where no channel exists; relay bounded hand-offs through the Chief instead.

## Red-tier review

Green and yellow work merges without human review. For red work, have the worker write an annotated walkthrough (intent, each changed file in reading order, risks, what to verify) and deliver it through the installed native Codex review interface if one exists; otherwise post it as a PR comment. When hosted in Herdr, use the Hunk split in `herdr.md`. Ask the human for attention and merge only after explicit approval of the current head.

## Check-ins and PR monitoring

Use the installed, documented Codex background-task/scheduler or approved integration to create one separately identified recurring check-in task per supervised session. Follow the registry, response deadlines, and GitHub event/cursor requirements in `coordination.md`. Maintain PR monitoring and keep workers available for revisions until verified merge to trunk or explicit rejection.

Confirm that the mechanism can wake the Chief repeatedly, including while idle. A command running in another terminal, a retained shell session, or a blocking agent wait is not proof of an autonomous supervision loop. `gh pr checks --watch` covers CI only, not all comments/reviews/CI history.

If recurring wake-up or cross-session messaging is unavailable, state the gap and ask for a supported scheduler/bridge or an explicitly approved manual supervision mode. Do not label manual checks as active autonomous monitoring. Do not start unmanaged `nohup`/detached loops and assume their output reaches the Chief.

## Recovery and disposal

On resume/restart, rediscover live worker handles and reconcile the durable registry with working state, git status, GitHub, and the actual background-task facility. Recreate only missing monitors and process unseen PR events. Never blindly reuse stale handles.

After terminal disposition, stop assignment-owned monitors/tasks, ask the worker to stop its owned servers/watchers, and verify exits. Shut down the Codex worker through its documented interface only when safe and no human input is pending. Do not assume Claude's `/exit` or Pi process IDs apply. Preserve evidence and apply the shared Worktrunk, dirty-tree, dependent-branch, and resource ownership safeguards.

Use verified project attribution conventions for the actual Codex runtime/model. Do not copy the Claude Code footer or fabricate a co-author identity.
