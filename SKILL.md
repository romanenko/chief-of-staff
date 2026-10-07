---
name: chief-of-staff
description: >-
  Coordinate worker agents through their full thread lifecycle: scope, delegate, supervise, review, shepherd artifacts into main or reject them, and clean up. Use when asked to act as chief of staff, coordinate work, deploy agents, delegate tasks, or manage a batch of assignments. Supports Pi, Claude Code, and Codex Harnesses through references.
---

# Chief of Staff

The **Chief agent** owns coordination and acceptance; **worker agents** own implementation. A **worker thread** is one assignment's durable context, worker session, isolated working state, artifact, and supporting resources. The **Harness** supplies execution, communication, and lifecycle controls.

## Coordination pattern

Keep the Chief in the coordinating thread. Delegate implementation—including small fixes and shipping mechanics—to workers. Run independent threads in parallel; sequence dependent threads explicitly. Maintain a durable thread registry with identity, scope, state, progress, artifact, permissions, dependencies, and owned resources.

## Worker-thread lifecycle

1. **Scope and assess risk.** Define the outcome, acceptance criteria, boundaries, verification, dependencies, and approval gates. Assign a preliminary risk tier (green/yellow/red) with the triggers that would raise it, per [operational procedures](references/operations.md). Give each coherent assignment one worker thread.
2. **Start and delegate.** Provision isolated working state, launch the worker through its Harness, verify identity/readiness, and send a self-contained spec that states the preliminary risk tier and escalation triggers. Confirm acknowledgement; do not assume shared conversation context.
3. **Supervise.** Establish one recurring check-in process per worker session. Track evidence-backed progress, answer blockers, and redirect when needed. Require milestone, scope-change, and risk-flag reports; idle or silence is not completion. A worker flag that raises the tier takes effect immediately.
4. **Review, revise, and route by risk.** Inspect the artifact and verification evidence and delegate corrections to the worker until acceptance criteria hold. Confirm the final risk tier against the actual diff. Green and yellow work proceeds to merge without human review. Red work requires a prepared human review through the Harness's review channel and explicit human approval of the current artifact.
5. **Shepherd.** On authorized PR creation, keep the worker available. Monitor all comments, reviews, and CI runs; assign follow-ups, verify fixes and current-head readiness, and shepherd the artifact into main under its risk route and repository protections. PR creation is not completion.
6. **Resolve and dispose.** Confirm merge into main or explicit rejection. Record the outcome, stop check-ins and supporting processes, terminate the worker, and remove disposable thread-owned working state and resources. Preserve essential evidence; verify cleanup and report leftovers.

On resume or failure, reconcile recorded state with live Harness sessions and artifact state before acting. Never bypass permissions, discard unknown work, or dispose of another thread's resources. If a required capability is absent, surface the blocker instead of claiming supervision is active.

## Hosting check

Detect Herdr with `test "${HERDR_ENV:-}" = 1`. Only on success, read [Herdr coordination](references/herdr.md); its Herdr and Hunk workflows are hosted-only. Otherwise—including pure Claude, Claude Code/CLI, Codex CLI, or desktop apps—ignore those workflows and use native Harness tools. If shell/environment access is unavailable, treat hosting as unconfirmed and skip them. Hosting is independent of Harness or model identity.

## References

Choose by the Chief's **Harness**, not its model/provider; read the matching communication reference before coordinating:

- **Pi:** [Pi communication, codemode, TypeSafe/Jev classifiers, and process controls](references/pi.md); use codemode and TypeSafe/Jev classifiers whenever appropriate.
- **Claude / Claude Code / Claude CLI:** [Claude communication and lifecycle controls](references/claude.md).
- **Codex:** [Codex communication and lifecycle controls](references/codex.md).

For mixed-Harness workers, read their reference too and verify the bridge.

For terminal-based PR/Actions watching, consult [gh-observer](references/gh-observer.md) when the extension is available. It is optional and independent of Herdr; verify PTY supervision and supplement its CI-only coverage with the shared PR-event monitoring contract.

Read [coordination details](references/coordination.md) for registry, check-in, PR-monitoring, recovery, and cleanup specifics. Consult [operational procedures](references/operations.md) before applicable setup, review, shipping, or disposal actions; it contains generic execution safeguards, risk assessment and routing, approval policy, and dependency handling. Resolve references relative to this skill directory.
