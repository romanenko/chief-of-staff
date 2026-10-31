# Chief of Staff

An agent skill for the **coordinator pattern**: a Chief scopes and delegates work, supervises independent worker threads, reviews evidence, shepherds artifacts into the target branch, and disposes of accepted or rejected work.

The workflow is Harness-neutral. Pi, Claude, and Codex have separate communication references. Optional terminal hosting, annotated review, and gh-observer PR/Actions watching are isolated in separate references, loaded only when applicable and available.

## Install

```sh
npx skills add romanenko/chief-of-staff
```

Follow the prompts to choose your agent and installation scope, then reload or start a new agent session.

## Usage

Ask your agent to act as chief of staff and describe the assignments, boundaries, and expected outcomes. For example:

```text
Act as chief of staff. Coordinate these assignments in parallel, verify the
results, and shepherd each change through review and integration.
```

In Pi, you can invoke the skill directly:

```text
/skill:chief-of-staff Coordinate the following assignments…
```

For other agents, use their supported skill invocation or ask for the chief-of-staff workflow in your prompt.

## Resources

- [Main skill](SKILL.md): brief coordination pattern and worker-thread lifecycle.
- [Shared lifecycle](references/coordination.md): registry, recurring check-ins, PR/CI supervision, recovery, and disposal.
- [Operations](references/operations.md): isolated execution, risk assessment and routing (green/yellow merge without human review; red gets a prepared human review), and dependent PR safeguards.
- [Pi](references/pi.md): intercom, managed background processes, codemode composition/parallelism, and risk assessment with the TypeSafe Jev classifier.
- [Claude](references/claude.md): capability-aware native delegation, messaging, and lifecycle controls.
- [Codex](references/codex.md): capability-aware native delegation, messaging, and lifecycle controls.
- [gh-observer](references/gh-observer.md): optional terminal/PTY PR and Actions watcher, supervisor requirements, current-head verification, and coverage limits; use when available, with or without Rex.
- [Rex coordination](references/rex.md): live target discovery, project sessions and assignment tabs, direct Worktrunk lifecycle, native worker launch/messaging, terminal events, annotated review, and ownership-safe cleanup. Supports both hosted and explicitly targeted external callers.

## Capabilities and limits

Use the target repository's documented setup, checks, and conventions. Choose approved models and preserve permissions; this skill imposes no fixed model, company, application, or package-manager configuration.

For Pi, load the installed `pi-intercom` and `pi-processes` integrations and enable `codemode` where supported. Use codemode and TypeSafe/Jev classifiers whenever appropriate to compose tool calls and make evidence-backed, finite-choice judgments. In Pi, risk tiers are assessed with codemode and Jev, both at preliminary assignment and in worker checks. Classifiers do not grant merge or deletion permission.

Every supervised worker needs a separately tracked recurring check-in task. PR supervision continues through revisions until verified merge or explicit rejection. PR creation, idle workers, and passing CI are not acceptance by themselves.

The skill defines monitoring behavior; it does not bundle an executable scheduler or GitHub monitor. Verify the chosen implementation can collect all relevant events and wake the Chief. If the Harness lacks a required capability, surface the gap rather than claiming autonomous supervision is active. Background work is not automatically durable across process exit, machine sleep, or restart.

Rex is optional for native CLI and desktop environments. When used, it hosts terminals rather than replacing the agent Harness. Use one project session with named assignment tabs; worktree identity and isolation come from Worktrunk/Git, not terminal nesting. Hunk is strictly opt-in: use it only when the human explicitly requests Hunk. Default to the Harness/PR walkthrough; neither Rex hosting nor red-tier work automatically opens Hunk.

Rex event streams and program-status indicators provide observation, not an agent broker, recurring scheduler, or full GitHub monitor. Keep those responsibilities in the verified Harness integrations. Do not use host-specific messaging auto-launchers: launch workers explicitly with Rex, then discover their native addresses.

Native CLI and desktop environments do not require hosted terminal workflows. Use only installed, documented interfaces; never invent cross-session messaging or substitute a permission-bypassing adapter.
