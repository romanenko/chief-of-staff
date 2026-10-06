# Claude coordination and communication

Use this reference for Claude, Claude Code, or Claude CLI, irrespective of the model name. Read `coordination.md` as well. Desktop/chat environments may expose fewer execution tools; capability checks still apply.

## Launch and model policy

Discover native delegation/session controls and confirm worker identity, working state, and readiness. Select models and worker roles from the user's approved configuration; do not impose fixed model preferences. If a configured model is unavailable, ask rather than silently substitute or weaken permissions. Use only launch flags documented for the installed native interface.

## Talking to agents

Use `ListAgents`/`SendMessage` only when the installed Claude integration exposes them and supports the intended worker sessions. Otherwise use documented native delegation/messaging; do not invent those tools or assume separate CLI processes can converse. If no channel exists, surface the gap and ask for a supported bridge or an approved manual hand-off.

When available, discover current addresses with `ListAgents`, record the exact worker identity, and send specs, review rounds, and ship prompts with `SendMessage` (`notify_when_idle: true` where supported). Put long specs in git-excluded `<path>/.review/SPEC.md` and send a pointer plus summary. Ask workers to reply to the sender of the received message with task ID, milestone, evidence, blocker, and next action.

Permit bounded peer hand-offs through the verified channel. The Chief owns sequencing, ship authorization, and risk; workers copy it on scope/dependency changes. Rediscover addresses after resume and confirm acknowledgements; neither delivery nor idle proves completion.

Use the available native artifact-review channel. Respect pending human input and approval dialogs through the native interface.

## Recurring check-ins and PR monitoring

Use an available, documented background-task or scheduler facility to maintain a separate check-in task for every supervised worker and monitor each PR per `coordination.md`. Idle notifications alone are not recurring check-ins. Verify that events can actually wake the Chief; a detached shell does not establish a supervision loop.

If tools cannot provide recurring wake-ups, report the gap and ask for a scheduler/bridge or an explicitly approved manual mode. Do not claim autonomous monitoring is active. Do not invoke Pi's `process` or `intercom` unless a verified integration exposes them. Desktop/chat access alone is not permission or capability to launch local workers.

## Exit and attribution

After terminal disposition and safety checks, stop owned tasks and exit the worker through its documented native interface. Protect pending user input and verify termination before deleting working state.

Use the repository's attribution policy and identify the actual runtime/model only when required. Do not fabricate co-author identities or attribute another Harness's work to Claude Code.
