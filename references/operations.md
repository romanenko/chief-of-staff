# Operational procedures

Use this reference for isolated execution, artifact review, risk assessment and routing, shipping, and dependency safeguards. Follow the target repository's documented commands and conventions; never assume a particular application, package manager, model, or issue tracker.

## Isolated execution and assignment

Create one isolated working state per coherent assignment using the approved worktree manager or Harness interface. If Worktrunk is selected, inspect its installed CLI and hooks, create worktrees from the primary checkout, and read returned paths rather than predicting them. Let the Chief own creation/removal; workers must not switch, merge, or remove working state without instruction. Do not bypass hook approvals.

Send a self-contained spec with:

- Outcome, acceptance criteria, allowed scope, and dependencies.
- Working path/branch, relevant files, and required project instructions.
- Exact verification commands, expected evidence, and review channel.
- Preliminary risk tier, its rationale, and the escalation triggers that would raise it.
- Progress/report format, escalation path, and the merge route that follows from the tier.

Keep disposable notes, screenshots, scripts, and logs in an assignment-owned, git-excluded location. Use the repository's documented setup and verification commands. Revert only known temporary changes; never overwrite unrelated edits. End verification with a status check showing only intended changes.

Review the actual diff and test evidence. Explain real problems and required corrections rather than unrelated stylistic preferences. Source reported quantities or label them as estimates.

## Shipping and acceptance

Assigning work to the Chief authorizes shipping it under the risk routing below, unless the user or repository sets a stricter policy. Workers perform commit, push, and PR mechanics; the Chief verifies the artifact and owns acceptance.

1. Specify intended files, commit/PR conventions, target branch, relevant issue links if required, and truthful runtime/model attribution where applicable. Never invent an identity or commit disposable review artifacts.
2. Run commit, push, and PR creation as separately inspectable steps. Respect permission denials; do not perform a denied operation through another session.
3. Verify the commit, clean working state, PR diff, target branch, metadata, and required verification. Route discrepancies to the worker before reporting readiness.
4. Confirm the final risk tier and follow [the shared lifecycle](coordination.md) for comments, reviews, CI, revisions, merge, and cleanup.

Merge according to the risk route and within repository protections. Green and yellow work merges without human review. Red work requires explicit human approval of the current artifact, even when a broader batch authorization exists. Deployment/release authority is separate from merge authority; do not infer it.

Confirmed merge or explicit rejection starts cleanup. Remove only assignment-owned resources, preserve essential evidence, check dependencies before deleting branches, and report blockers instead of forcing disposal.

## Risk assessment

Risk is assessed twice by the Chief and continuously by the worker:

1. **Preliminary tier (before delegation).** From the outcome, expected files and surfaces, dependencies, and rollback path, the Chief assigns a tier and lists the triggers that would raise it (for example, touching auth, payments, schemas, or customer data). The tier and triggers go in the spec, so the worker knows from the hand-off whether the change is low risk.
2. **Worker flag (during implementation).** The worker checks its own work against the assigned tier and triggers. If the work hits a trigger or otherwise looks riskier than assigned, it reports a risk flag to the Chief immediately with the evidence, before continuing past that point. A worker can raise the tier but never lower it.
3. **Final tier (before merge).** The Chief assesses the actual diff and verification evidence. The final tier is never lower than the preliminary tier or any accepted worker flag unless the Chief records why the earlier concern does not apply.

In Pi, the Chief and Pi workers make these assessments with codemode and the TypeSafe Jev classifier, as described in `pi.md`. Every Harness uses the same rubric and routing.

Use the repository's rubric when provided. Otherwise use these general categories:

- **Low / green:** negligible plausible harm, such as bounded documentation, test, or internal tooling changes with no sensitive execution path affected.
- **Moderate / yellow:** bounded plausible harm involving authentication, payments, sensitive data, external integrations, or user-facing behavior.
- **High / red:** material or broad harm, destructive/schema changes, difficult rollback, or wide impact without effective safeguards.

Assess worst plausible harm and blast radius, not just file count or a passing success path. When evidence is incomplete or categories are close, collect more evidence or choose the more cautious category.

Record the category, affected paths, plausible failure/cost, tests and mitigations, remaining uncertainty, rollback considerations, and concrete post-change signals. If classifier-assisted, record the provider/model, criteria, proposed result, and Chief's final decision. Do not claim a classifier ran when it did not.

Use existing repository labels/comment formats where available. Do not impose a foreign scoring workflow or create labels without authority. Reassess material changes, validate readiness against the current head, and re-route when the tier changes.

## Risk routing

| Final tier | Route |
| --- | --- |
| Green / yellow | Merge without human review once current-head required checks pass, actionable feedback is resolved, and repository protections are satisfied. Report the merge with its tier and rationale. |
| Red | Do not merge. Have the worker write an annotated walkthrough of the change, prepare the review through the Harness's review channel (Herdr/Hunk split when hosted; otherwise the channel in the Harness reference), and ask the human for attention. Merge only after explicit approval of the current head; renew approval if the head changes materially. |

If repository protections require a human approval the Chief cannot supply, report the PR as awaiting required approval; never bypass protections. The walkthrough explains intent, how each change achieves it, risks, and what the reviewer should verify. Keep secrets and customer data out of review artifacts.

## Dependent PR safeguards

- One coherent feature, one PR. Changes sharing acceptance criteria or a deployment gate belong together.
- Stack separate dependent features only when useful and approved. Each PR should show only its own change; the Chief sequences the work and workers perform rebases.
- Work bottom-up. Verify actual remote branch tips before rebasing; do not rely solely on potentially stale PR metadata.
- Use the installed stack tool's documented workflow. Avoid checkout-based synchronization when branches are already checked out in separate worktrees.
- After a base merges, rebase/retarget dependent PRs and verify their state before deleting the base branch. Never delete a branch another PR still targets.
- Use `--force-with-lease` for authorized rewritten branches. Preserve uncommitted work through an explicit temporary checkpoint; do not use a shared stash without coordination.
- After conflict resolution, re-read affected files, run whitespace/conflict-marker checks, and re-run verification. Inspect affected commit trees as well as the tip when rewriting history.
- Refresh stale PR descriptions after rebases. Prefer stable references and avoid rewriting unrelated identifiers.
- Investigate failures with repeatable evidence. State hand-off hold points upfront and verify merged behavior rather than relaying outdated descriptions.
- Make checks fail visibly; never turn errors into a “clean” result. Have the receiving worker apply and verify explicit patches when transferring changes.

## Ownership and evidence

Never dispose of resources or working state you did not create or were not explicitly authorized to manage. Rediscover live identifiers after recovery; do not guess paths or session handles. Research workers should save large reports in an assignment-owned location and return a path plus concise findings. Preserve human input, approval decisions, and unrelated work throughout the lifecycle.
