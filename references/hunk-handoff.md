# High-risk agent handoff

The chief copies the instructions below into the agent’s initial task prompt, replacing `<path>`, `<base>`, `<agent-pane>`, and `<name>`. `<base>` is the recorded starting commit, not a moving branch. Keep the task’s risk explanation and acceptance criteria alongside this handoff.

## Instructions for the agent

Your work requires human review. Finish implementation and verification, then prepare the annotated review below. Keep your session, worktree, and review open for feedback. Do not merge or clean up before approval.

1. **Learn the installed tools.** Load the Herdr skill. Run `hunk skill path hunk-review` and read the returned file. It matches the installed Hunk version; the [full Hunk documentation](https://www.hunk.dev/llms-full.txt) provides further details. This assignment authorizes opening the user’s Hunk window through Herdr. Launch it in a separate pane, then control it with `hunk session *`; do not run an interactive Hunk command in your own execution tool.
2. **Open the whole task diff.** Run `herdr pane split --pane <agent-pane> --direction right --cwd <path> --no-focus`. Read `<review-pane>` from the response’s `.result.pane.pane_id`, then run `herdr pane run <review-pane> "hunk diff <base> --agent-notes"`. Compare with the starting commit so committed, staged, unstaged, and new files are covered. `HEAD` would hide commits you made. Keep temporary notes and screenshots out of the reviewed changes and commits.
3. **Inspect before annotating.** Wait for `hunk session get --repo <path> --json` to find the review, then run `hunk session review --repo <path> --json`. Verify that all intended files are present. Request `--include-patch` only when needed. If multiple sessions match, select the correct session ID explicitly for subsequent commands.
4. **Guide the reader.** Add a few inline notes in a useful reading order: what changed and why, decisions needing approval, what could fail and who is affected, and verification or remaining gaps. Use real relative file paths and 1-based lines from the loaded diff. Each note needs exactly one anchor; use `oldLine` for removed code. Apply a JSON batch with `hunk session comment apply --repo <path> --stdin`. The example below shows the shape; replace its file, line, and text with actual findings.

   ```json
   {"comments":[{"filePath":"src/example.ts","newLine":42,"summary":"Decision to review.","rationale":"Reason, possible impact, and verification.","author":"<name>"}]}
   ```

   Send this batch through stdin; it is not the `--agent-context` sidecar format. Use plain text notes. Add precise warning highlights only when useful, following the installed skill. Confirm notes with `hunk session review --repo <path> --include-notes --json`. Preserve the user’s focus.
5. **Report and stay available.** Tell the chief “ready for review,” with workspace, review pane/session, verification results, and key decisions. A missing tool, failed session, or rejected annotation means the review is incomplete: report the blocker and preserve the work.
6. **Handle feedback.** Read `hunk session comment list --repo <path> --type all --json` before changing code or reloading, and retain unresolved user notes. Answer with `hunk session comment add --repo <path> --reply-to <id> --summary "..." --author <name>`. After fixes, rerun verification and `hunk session reload --repo <path> -- diff <base> --agent-notes`. Reloads can drop notes on changed files: inspect what remains, refresh anchors, and reapply missing agent notes without duplicates. Preserve user feedback and report any lost anchors. Keep the review open.
