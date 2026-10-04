# Chief of Staff

An agent skill implementing the **coordinator pattern**: one agent plans and reviews, while other agents make the changes.

The chief gives workers detailed plans and reviews their results. Defaults are `openai/gpt-6-astra` with medium reasoning for the chief and `openai/gpt-6.1-sol` with medium reasoning for workers. Pi workers communicate through [pi-intercom](https://pi.dev/packages/pi-intercom); Herdr organizes their workspaces and reviews, and Worktrunk isolates each task. Explicitly requested non-Pi workers use a supported messaging fallback.

## Install

Use [Pi](https://pi.dev) inside [Herdr](https://herdr.dev), with [Worktrunk](https://worktrunk.dev) and [Hunk](https://www.hunk.dev) installed. Install pi-intercom in Pi:

```bash
pi install npm:pi-intercom
```

Restart or reload the participating Pi sessions so the extension is loaded. Then ask Pi:

> Install the chief-of-staff skill from https://github.com/romanenko/chief-of-staff into ~/.agents/skills/chief-of-staff. Also load Herdr’s skill if needed.

The local skill directory should contain `SKILL.md` and the complete `references/` directory. Pi discovers `~/.agents/skills/`; run `/reload` after updating it.

To start in Pi:

```text
/skill:chief-of-staff Help me with [your task].
```

## Coordination

The chief discovers connected workers, sends detailed task plans, answers blocking questions with threaded replies, and verifies their results. Workers send progress and completion asynchronously; delivery alone never proves completion. See [the coordination protocol](references/intercom-coordination.md) for targeting, timeout/cancellation handling, handovers, subagent escalation, and non-Pi fallbacks.

Every worker workspace is opened with `herdr worktree open --workspace <chief-workspace-id> --path <worktree-path> --label "<task>" --no-focus` after Worktrunk creates the worktree. This registers repository metadata so Herdr’s Spaces view groups workers under the main repository workspace, rather than showing unrelated flat workspaces. Worktrunk still owns worktree creation/removal; Herdr owns the visible layout. The chief verifies grouping before launch and preserves user focus. This is repository/worktree grouping, not an agent parent-child relationship.

High-risk changes still require [annotated Hunk review](references/hunk-handoff.md) and user approval before integration. Worktree ownership, isolation, and safe cleanup are unchanged.

Codex remains supported when explicitly selected; invoke the skill there with `$chief-of-staff` and use the documented fallback rather than assuming Codex has the Pi intercom tool.
