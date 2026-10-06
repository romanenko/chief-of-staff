# Chief of Staff

An agent skill for the **coordinator pattern**: a Chief scopes and delegates work, supervises independent worker threads, reviews evidence, shepherds artifacts into the target branch, and disposes of accepted or rejected work.

The workflow is Harness-neutral. Pi, Claude, and Codex have separate communication references. Optional terminal hosting, annotated review, and gh-observer PR/Actions watching are isolated in separate references, loaded only when applicable and available.

## Install

Copy `SKILL.md` and the entire `references/` directory into the skill location supported by your Harness. Do not copy Git metadata or install individual reference files without the main skill.

For Pi, use a skill directory such as `~/.agents/skills/chief-of-staff/`, reload the session, and invoke:

```text
/skill:chief-of-staff Coordinate the following assignments…
```

For other Harnesses, follow their installed skill-loading interface. Do not assume a model/provider name identifies the Harness.

## Resources

- [Main skill](SKILL.md): brief coordination pattern and worker-thread lifecycle.
- [Shared lifecycle](references/coordination.md): registry, recurring check-ins, PR/CI supervision, recovery, and disposal.
- [Operations](references/operations.md): isolated execution, risk assessment, approval policy, and dependent PR safeguards.
- [Pi](references/pi.md): intercom, managed background processes, codemode composition/parallelism, and TypeSafe/Jev classification.
- [Claude](references/claude.md): capability-aware native delegation, messaging, and lifecycle controls.
- [Codex](references/codex.md): capability-aware native delegation, messaging, and lifecycle controls.
- [gh-observer](references/gh-observer.md): optional terminal/PTY PR and Actions watcher, supervisor requirements, current-head verification, and coverage limits; use when available, with or without Herdr.
- [Terminal hosting](references/herdr.md): optional hosted coordination and annotated review, loaded only when the main skill's hosting check succeeds.

## Capabilities and limits

Use the target repository's documented setup, checks, and conventions. Choose approved models and preserve permissions; this skill imposes no fixed model, company, application, or package-manager configuration.

For Pi, load the installed `pi-intercom` and `pi-processes` integrations and enable `codemode` where supported. Use codemode and TypeSafe/Jev classifiers whenever appropriate to compose tool calls and make evidence-backed, finite-choice judgments. Classifiers do not grant merge or deletion permission.

Every supervised worker needs a separately tracked recurring check-in task. PR supervision continues through revisions until verified merge or explicit rejection. PR creation, idle workers, and passing CI are not acceptance by themselves.

The skill defines monitoring behavior; it does not bundle an executable scheduler or GitHub monitor. Verify the chosen implementation can collect all relevant events and wake the Chief. If the Harness lacks a required capability, surface the gap rather than claiming autonomous supervision is active. Background work is not automatically durable across process exit, machine sleep, or restart.

Native CLI and desktop environments do not require hosted terminal workflows. Use only installed, documented interfaces; never invent cross-session messaging or substitute a permission-bypassing adapter.
