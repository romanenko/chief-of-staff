# Chief of Staff

An agent skill implementing the **coordinator pattern**: the chief plans and reviews while workers make isolated changes. [Pi Babysit](https://github.com/yusukeshib/pi-babysit) is the shared substrate for agent coordination **and** asynchronous operations—not just a messaging replacement.

Defaults are `openai/gpt-6-astra` with medium reasoning for the chief and `openai/gpt-6.1-sol` with medium reasoning for workers, verified against the installed model catalog. Worktrunk isolates editing tasks. Herdr provides visible workspaces and annotated Hunk reviews when needed; ordinary background Pi workers live in the Babysit widget without requiring a separate Herdr agent.

## Install

Use [Pi](https://pi.dev) with [Worktrunk](https://worktrunk.dev). Install the Pi extension and its separate supervisor binary:

```bash
pi install npm:@yusukeshib/pi-babysit
cargo install --git https://github.com/yusukeshib/babysit
babysit --version
```

The documented extension requires **babysit 0.13.0 or newer** and does not auto-install it. A [prebuilt release](https://github.com/yusukeshib/babysit/releases) on `PATH` is an alternative to Cargo. Reload Pi after installing/updating the extension and verify the `babysit_*` tools are loaded. This workflow does not require the retired peer-messaging extension.

For visible terminal/Hunk review workflows, also install [Herdr](https://herdr.dev) and [Hunk](https://www.hunk.dev), run the chief in Herdr, and load Herdr's installed skill (`herdr --skill`). High-risk work still requires this annotated-review capability and user approval; its absence is a blocker, not permission to skip review.

For GitHub PR supervision, install/authenticate `gh` and install [gh-observer](https://github.com/fini-net/gh-observer):

```bash
gh extension install fini-net/gh-observer
gh extension list
gh observer --help
```

Do not print credentials. Then ask Pi:

> Install the chief-of-staff skill from https://github.com/romanenko/chief-of-staff into ~/.agents/skills/chief-of-staff, including its complete references/ directory.

The skill directory must contain `SKILL.md` and the complete `references/` directory. Pi discovers `~/.agents/skills/`; run `/reload` after updating it. Resolve references from that skill directory, not the current repository checkout.

```text
/skill:chief-of-staff Help me with [your task].
```

## What the chief supervises

| Work | Babysit mechanism |
| --- | --- |
| Research, implementation, diff review | Reusable Pi subagents; bounded budgets, parallel launch, explicit result collection |
| Worker decisions and revisions | Chief collects `BLOCKED`/phase results, steers active workers, sends tasks to settled workers |
| Builds, tests, setup, services, installers | Supervised processes; exit notifications, readiness waits, bounded logs, PTY interaction |
| PR CI and standalone Actions runs | PTY-backed background `gh observer <URL>`, with metrics/startup handling and Babysit exit notifications |
| Repository CI overview | Persistent `gh observer --repo=OWNER/REPO` in a Babysit PTY; explicitly stopped when no longer needed |
| Reviews/comments, changed heads, conflicts, merge/close | Chief verifies at phase boundaries and routes repairs; autonomous late-event notification requires an authorized event source |
| Explicitly selected non-Pi workers | Supervised native CLI runs/resumes with collected output/artifacts |

See [the coordination protocol](references/babysit-coordination.md) for identities, task contracts, worktree targeting, asynchronous scheduling, reuse, failure recovery, and cleanup. See [PR supervision](references/pr-supervision.md) for the concrete opening → watch → review/fix → reverify → rearm → authorized integration loop.

### Important semantics

- Babysit is **not a peer-to-peer message broker**. The chief communicates with its own worker sessions via `babysit_send`, reads output, collects results with `babysit_wait`, and relays dependencies. Workers do not invent send/ask/reply calls back to the chief.
- Background **processes** automatically notify on exit. Background **subagents** still require explicit collection before the parent finishes. A process exiting and an agent task settling are different events.
- The current subagent tool has no cwd override. Reusable workers need explicit absolute worktree file paths and command targeting. For genuine worktree startup/project configuration, launch the native agent at that cwd as a supervised process.
- Real Pi shutdown kills its supervised work. Reload/resume of the same session reconnects to it; detached does not mean an always-on daemon. Longer-lived supervision needs a separately authorized external service.
- `gh-observer` needs a PTY for a live watch; non-terminal output takes a single snapshot that can exit zero while checks are pending. Repo mode is a persistent CI overview, not an auto-exiting completion/event feed for every PR or human comment.
- Passing checks, delivered steering, a settled worker, and an open PR are not approval or integration. Observers are read-only; they never auto-merge or bypass protections.

For task-specific visible/Hunk workspaces, open the Worktrunk-created checkout with `herdr worktree open --workspace <chief-workspace-id> --path <worktree-path> --label "<task>" --no-focus`. Verify repository grouping and preserve focus; Worktrunk still owns creation/removal. A headless worker receives an explicit review-host pane, not an assumed inherited pane.

High-risk changes require [annotated Hunk review](references/hunk-handoff.md) and explicit user approval of the current reviewed diff before integration. Changes after approval require renewed review/approval. Preserve dirty/unmerged work and close only task-owned resources.

## Verified interfaces

The operational protocol was checked against pi-babysit 0.6.18, babysit 0.14.6, and installed gh-observer v4.1. Consult the installed README, tool schema, and CLI help after upgrades rather than assuming unsupported flags, cwd overrides, messaging semantics, or watch/event capabilities. PR supervision uses the selected extension directly; no custom Bash watcher is bundled.
