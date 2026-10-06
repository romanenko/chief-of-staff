# Pi coordination and communication

Use this reference when the Chief runs in the Pi harness, irrespective of its model/provider. Read `coordination.md` for the shared lifecycle contract.

## Contents

- [Capability checks and worker launch](#capability-checks-and-worker-launch)
- [Codemode and TypeSafe/Jev classifiers](#codemode-and-typesafejev-classifiers)
- [Direct communication](#direct-communication)
- [One managed check-in process per worker](#one-managed-check-in-process-per-worker)
- [PR monitor](#pr-monitor)
- [Recovery, exit, and disposal](#recovery-exit-and-disposal)

## Capability checks and worker launch

Read the installed `pi-intercom` and `pi-processes` skills before use. Verify `intercom({ action: "status" })`, discover peers with `intercom({ action: "list" })`, and inspect managed processes with `process({ action: "list" })`.

Launch Pi workers in isolated working state via a supported Pi delegation/session interface. Select the user-approved model/provider and respect its permissions. Confirm cwd, branch, and intercom registration before delegation. Name the Chief and workers distinctly with `/alias` through the supported session interface.

Pi intercom reaches registered Pi sessions, not arbitrary Claude or Codex sessions. For a mixed-runtime worker, load that worker's reference too and verify a supported communication bridge. Report unsupported direct messaging rather than claiming delivery.

## Codemode and TypeSafe/Jev classifiers

**In Pi, use the `codemode` tool whenever appropriate, and pair codemode with TypeSafe/Jev classifier models whenever appropriate.** This applies to both the Chief and Pi workers. The installed tool is named `codemode` (not `codemod`).

Read the installed Pi codemode/model documentation before using its APIs. Prefer codemode for composing dependent tool calls, parallelizing independent calls, and filtering/aggregating large results before returning them to the model. Examples include collecting multiple workers' progress, reading related files, reconciling PR comments and CI, and turning evidence into a finite-choice decision.

- Use `Promise.allSettled()` for independent calls so partial failures remain visible; await all required work before the script ends. Sequence calls when later inputs depend on earlier results.
- Never parallelize conflicting writes, branch operations, approval-dependent actions, or calls sharing mutable state. Check returned error/status fields as well as rejected promises; prior tool effects are not rolled back on script failure.
- Return concise evidence, IDs, errors, and decisions rather than full logs. Keep durable thread state in the registry, not solely in script-local variables.
- Use direct tools for simple one-off calls where scripting adds no value. Do not force every task into a script or a classifier.
- Codemode has no direct filesystem/network/timers: use `tools.*`. It cannot call itself. Use managed processes for recurring supervision, not a long-lived codemode loop.

### Bounded judgment and routing

Use TypeSafe/Jev through codemode for judgment that fits typed questions: risk category, urgency, feedback ownership, or choosing the best eligible next step from a finite set. Define explicit labels and criteria; provide the actual diff, tests, dependencies, and approval state as relevant evidence. Apply deterministic eligibility/permission checks first so prohibited actions are not candidates.

Discover usable models with `models.getAvailableOfType("classifier")` and select a TypeSafe/Jev entry. Provider and model IDs vary; do not assume credentials or hardcode an unavailable ID. Then call `models.classify(model, { state, questions })`, using `choice`, `bool`, or `score` questions. For example, after collecting evidence:

```js
const available = await models.getAvailableOfType("classifier");
const jev = available.find(m =>
  m.provider === "typesafe" && m.id.startsWith("jev")
) ?? available.find(m => /jev/i.test(m.id));
if (!jev) return { status: "classifier-unavailable", next: "Chief assessment" };

const evidence = load("riskEvidence"); // collected diff/test evidence, not a PR-title guess
if (!evidence) return { status: "missing-evidence", next: "Collect diff and tests" };
const result = await models.classify(jev, {
  state: evidence,
  questions: {
    risk: {
      type: "choice",
      instructions: "Assess worst plausible business harm using these criteria; choose the higher category when uncertain.",
      criteria: {
        green: "Negligible harm; no customer, revenue, auth, or sensitive-data path.",
        yellow: "Bounded plausible harm on checkout, attribution, messaging, third-party data, or auth.",
        red: "Wide/material harm, schema migration, or unflagged change affecting every visitor."
      }
    }
  }
});
if (result.stopReason !== "stop")
  return { status: result.stopReason, error: result.errorMessage };
return { provider: result.provider, model: result.model, risk: result.answers.risk };
```

Validate answer type, allowed label, confidence/probabilities, and evidence before using a result. Record classifier/model, criteria, result, and the Chief's decision in the thread registry. For routine eligible routing, a validated choice may select the next path; for risk assessment, the Chief checks it against the full rubric in `operations.md` and owns the final label/rationale. Classification is not human approval or permission to merge, delete, or expand scope.

On absent credentials, service errors, ambiguous/low-confidence output, or insufficient evidence, collect more evidence or have the Chief decide/escalate explicitly. Never silently classify an error as green or claim a classifier ran when it did not. Keep sensitive material out of classifier inputs unless its use with that provider is authorized.

## Direct communication

Use session IDs from a fresh roster; skip self. Re-list before reusing an ID after reconnect/resume. Use `cwd` as a safety guard and never infer identity from UI location, truncated names, or roster position.

```typescript
intercom({ action: "send", to: "<worker-id>", cwd: "<worktree-path>", message: "<complete spec or follow-up>" })
```

Use `send` for assignments, check-in requests, progress, completion, and review/ship prompts. Require an explicit worker reply containing the task ID and evidence: transport delivery is not task acceptance or completion. For long specs, use a git-excluded `<worktree>/.review/SPEC.md` and send a pointer plus summary.

Workers reply to the sender of the received message. Use `ask` only when a decision blocks progress; it requires a live peer, permits only one pending ask per sender, and has a timeout. The Chief uses `reply` for an inbound ask, with `pending` and `to`/`replyTo` when disambiguation is needed. Do not hold the Chief in a long `ask` while supervising other sessions. Timeout is not cancellation; inspect message state before retrying and use explicit cancellation/supersession when appropriate.

Permit peer messaging for bounded hand-offs; keep the Chief copied on scope/dependency changes. Use `handover` for transferring supervision with the registry path, outstanding decisions, monitor ownership, and PR state; do not create two active Chiefs for one assignment.

Keep intercom's inbound-trigger configuration compatible with autonomous supervision. `inboundTrigger: never` or delayed human-first delivery can prevent timely check-ins. Use a shared scope if configured; isolated scopes cannot communicate. Never set one machine-global stable ID for all workers. Broker mail is bounded runtime state, not the durable registry.

Use native interfaces for launch, human dialogs, artifact review, and exit. Protect actual pending human input.

## One managed check-in process per worker

Start a separate Chief-owned process named `chief-checkin-<task-id>` for every supervised worker. The command must be an actual tested monitor script, not an invented tool or a pseudocode path. It can emit a periodic `CHIEF_CHECKIN <task-id>` marker; the Chief wakes, rediscovers the worker, sends the progress request, and updates the durable registry. A scripted intercom CLI can be used only after checking the installed CLI interface; ordinary shell scripts cannot call the model's `intercom` tool.

Configure a log watch with `repeat: true`, `on: "turn"`, and a task-specific pattern so every interval wakes the Chief. Use `onFailure: "turn"` and `onKilled: "turn"` to catch loss of supervision. Record the returned opaque process ID, not only its name. Avoid duplicate starts by listing first.

The monitor script may schedule intervals internally; the Chief must not sleep, poll `process output`, or hold its turn after `process start`. End the turn or do other work and let notifications wake it. Check replies and overdue deadlines per `coordination.md`. A heartbeat request is not measured progress.

## PR monitor

On PR creation, start a Chief-owned managed monitor named `chief-pr-<task-id>` using a tested GitHub monitor command implementing the shared pagination/cursor contract. Use repeatable task-specific event markers such as `CHIEF_PR_EVENT <task-id>` and failure markers, with `on: "turn"`. Retain the worker's separate check-in process through PR revisions.

When installed and a PTY-capable supervisor is available, prefer [gh-observer](gh-observer.md) for PR/Actions watching; do not assume `process start` provides a PTY. `gh pr checks <n> --watch` is useful as an additional CI-only process, not a replacement for comment/review/full-run monitoring. Persist snapshot/cursor state outside the disposable worktree. Do not claim monitoring is active until the command runs successfully and its wake-up path is verified. This reference specifies behavior; it does not itself install a monitor script.

Use `process output` for targeted diagnosis and `read` on returned stdout/stderr paths for deep logs. Change noisy watches with `process update`, not a restart. Failures must remain visible; do not suppress them as successful empty snapshots.

## Recovery, exit, and disposal

On resume, reconcile intercom, processes, working state, registry, and GitHub before sending instructions. Do not apply Claude's address-rotation assumptions to Pi; rediscover actual identity and liveness.

Use `process stop` with recorded IDs for Chief-owned monitors. Ask workers to stop their own owned processes and report the results: do not assume the Chief's process list includes another session's resources. Use `process clear` only when all finished entries it would clear are safe to discard; it is not a task-scoped deletion operation.

Exit Pi workers using the installed native Pi shutdown mechanism after checking pending human input and worker state. Verify exit before worktree removal. Do not assume Claude's `/exit` command applies to Pi. Preserve transcript/evidence and use the shared Worktrunk/branch cleanup guards.

Attribute commits/PRs to the runtime and model that actually wrote them, using verified project conventions; do not stamp Pi workers with the Claude Code footer or invent a co-author email.
