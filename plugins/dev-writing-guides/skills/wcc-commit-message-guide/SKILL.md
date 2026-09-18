---
name: wcc-commit-message-guide
description: Guide and review checklist for a commit subject & body - state the decision and the high-level change with no restated technical detail, line-by-line worth, sync with the diff, and format/ID hygiene. Use when drafting a commit message, reviewing one, or before running git commit. Args - [sha | sha-range] (default - the staged change, or HEAD if the tree is clean).
---

Write or review **the commit subject and body**. The commit records why the change exists. The PR description addresses the reviewer, and the code comment addresses the next reader of that line. The commit addresses someone reading `git log` months later with no session and no ticket open.

**State what the change decides and what it changes at a high level. Never restate technical detail.**
That is the whole rule. Every check below is a way of applying it.

## Scope

- No arg + staged change → draft the message for `git diff --staged`.
- No arg + clean tree → review `HEAD`: `git log -1 --format=%B` and `git show HEAD`.
- Arg is a SHA → review that commit: `git log -1 --format=%B <sha>` and `git show <sha>`.
- Arg is a range → review each commit's message in `git log <range>`.

Always read the diff the message describes. A message cannot be judged without it.

If a `simple-english` skill is available, load it when drafting or rewriting. Do not wait to be asked.

## Check 1 — the decision and the high-level change

Keep three things at most. Everything else is a finding.

1. **The decision, and what forced it.** The wrong behavior or the gap the change answers.
2. **What the change does, at a high level.** One sentence, in terms of purpose, not mechanism.
3. **At most one fact the reader cannot get from the diff and would not expect from the subject**: a behavior change wider than the subject promises, deleted tests, a deferred part.

Cut every one of these, always:

| Cut | Because |
|---|---|
| Verification: tests pass, lint, mypy, coverage counts | Belongs in the PR description, never the commit |
| File lists, handler names, field lists, helper mechanics, the reason behind a query | The diff shows it |
| Fixture and mock mechanics (mock setup, export lists) | Test-harness detail, not a reason the commit exists |
| History archaeology: which earlier commit did what, what it assumed | `git log` and the tracker issue hold it |
| Anything the type and scope already imply ("Comments only. No behavior change." on a `docs()` commit) | Restates the subject |
| A rationale that already exists as a code comment in this diff | Stated twice |
| Restating the subject line in the first body sentence | Adds no fact |

Then apply these:

- **Judge sentence by sentence, and ask if the line is worth keeping.** Do not pass a paragraph whole. A sentence that restates a neighbor is a finding even when the rest of the paragraph is good.
- **No invented wording.** Every term that names a mechanism must already exist in the code or the repo docs. Grep for what the code calls it. If no name exists, describe the mechanism in plain words. No idiom, no analogy, no new coinage.
- **Shorten by deleting, not compressing.** A shorter but unreadable sentence is a new finding, not a fix. If the tighter rewrite lost meaning, keep the original.
- **Length gate.** Body is one or two paragraphs, 3 to 8 non-blank lines. A large feature commit can reach 12. Over 12 is a finding: cut whole items, or move the detail to the PR description or the tracker issue.
- **Small commit, small body.** A `test()`, `docs()`, or comment commit lands at 3 lines.
- **A part left for later gets one line**, not a paragraph ("The Alembic revision and the tests follow separately.").
- **The number of changes in the commit is not what the message is judged on.** A commit that carries several purposes is still worth avoiding, and a message forced to explain two unrelated purposes is a sign of one. But do not pad or split the message over the count.

## Check 2 — sync with the diff

- **Every claim matches the code**: behavior, identifiers, defaults, which paths change. Verify against the diff, not against memory of the session.
- **Naming drift**: terms in the message must be the identifiers actually in the diff.
- **Coverage**: the message covers what the commit contains, and nothing it does not. Re-verify after any `--amend` that changed the staged files.

## Check 3 — format & style

- **Subject**: `<type>(<scope>): <description>`, imperative, lowercase, no period, 72 characters or fewer. Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.
- **Subject says what the change does**, not what the defect was called.
- **Recurring commits keep the repo's established subject form.** Check `git log --oneline` first (for example `docs: reconcile docs with code through <sha>`).
- **Wrap the body at 72 mechanically, never by eye.** Rewrap with a script that keeps the subject and the trailers on their own lines. Never split an identifier to fit: restructure the sentence instead.
- **No em-dash anywhere.** Use a colon, a comma, a period, or parentheses.
- **No semicolons, no contractions, no present perfect.** Active voice, and name the actor. Banned modals: `should`, `would`, `could`, `may`, `might`. Use `can`, `will`, `must`.
- **No IDs in the message.** See Check 4.
- **Trailers**, last, in this order:

  ```
  Co-Authored-By: <the model identity for this session> <noreply@anthropic.com>
  Signed-off-by: <from git config, added by `git commit -s`>
  ```

  Do not type the `Signed-off-by` line. Pass `-s` and git writes it from `user.name` and `user.email`.

- **Commit from a file**: `git commit -s -F <file>` or `git commit -s --amend -F <file>`, with the file in the scratchpad directory. Not `-m`, and no heredoc holding apostrophes.

## Check 4 — ID hygiene

Nothing in this list stays in a commit message. There is no fixture-citation exception here: that one applies to code comments only.

- **Issue tracker IDs (`ABC-123`): delete.** No backfill needed. The branch name (for example `feature/abc-123-…`) and the PR already carry the ticket, so the message adds nothing.
- **Session IDs, cloud log-stream names, tracing or observability trace IDs, PR numbers, customer names or identity: move to the linked issue.** These are not derivable from the branch, so the information has to land somewhere before it leaves the message.

For each ID to move:

1. Find the linked issue from the branch name or the PR.
2. Check whether the issue's description or comments already hold it.
3. If missing, draft an issue comment. Follow any tracker-formatting constraints the repo documents in its CLAUDE.md or AGENTS.md. Show the draft, and post only on explicit approval.
4. Only then remove it from the message.

No linked issue → flag the ID and stop. Do not invent a place to put it.

Self-check before you show the draft:

```sh
awk 'length>72{printf "%d(%d): %s\n", NR, length, $0}' "$F"; echo "(nothing above = wrap ok)"
grep -nE "should|would|could|may |might|;|'ll|'re|has been|have been|—|[A-Z]{2,}-[0-9]+" "$F" || echo "clean"
```

## Output

**Draft mode**: show the full subject and body in chat, then stop. Run `git commit` only after the user approves the text. Do not commit first and offer to shorten afterward.

**Review mode**: a verdict per sentence, most-actionable first, then the proposed replacement message in full. Amend only on approval. For a follow-up fix on an unpushed commit, propose both `--amend` and a new commit, and let the user choose.

| sentence | verdict |
|---|---|
| "The gate returns early when info.json has no timestamp." | keep, this is the why |
| "No test reached that branch, so the reader lost a line of coverage." | keep the first half, cut the coverage count |
| "The test asserts that the download still runs." | cut, the diff shows it |
| subject, 73 characters | over the 72 limit, rewrap |
| body ¶2 cites `ABC-123` | delete, the branch name carries it |
| body ¶3 names a customer | already in the issue → delete; else backfill then delete |

End with one line of net body volume: lines before, lines after (for example "12 lines to 5"). Say what you cut and why, in one short list.

If the commit is already pushed and the finding is only a wording nuance, report it as "minor, no action" instead of proposing a rewrite that needs a force-push.
