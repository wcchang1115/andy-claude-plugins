---
name: wcc-code-comments-guide
description: Guide and review checklist for the current diff's added/changed code comments & docstrings - comment discipline and ID hygiene. Use when writing or reviewing code comments, or before committing a change that adds comments. Args - [base-ref] (default - working diff; clean tree → whole branch vs origin/main).
---

Review **the comment and docstring lines this change added or modified**, plus **any pre-existing comment adjacent to a changed code hunk that the change has made inaccurate**. Do not review code logic itself, and leave unaffected pre-existing comments alone — but DO read the surrounding code: verifying a comment's accuracy requires it.

## Scope

- No arg + dirty tree → `git diff && git diff --staged`
- No arg + clean tree → `git diff $(git merge-base origin/main HEAD)...HEAD` (whole branch/PR)
- Arg → always a base-ref: `git diff $1...HEAD`. To review a single commit, pass `<sha>^`.

Consider added/changed lines that are comments (`#`, `//`, `/* */`) or docstrings, plus any pre-existing comment within a few lines of a changed hunk — the latter checked for **accuracy only** (did the code change strand it?).

## Check 1 — comment discipline

Treat comments like production code: minimal, accurate, why-not-what.

- **Default to no comment.** Write one only when removing it would confuse a future reader — a hidden constraint, a non-obvious workaround, a subtle invariant. Don't restate what the code already shows, or behavior that's already obvious or verified.
- **Comment the why, not the what.** Explain non-obvious decisions and invariants; never narrate the next line.
- **Multi-line comments: judge each sentence.** Default budget is one line. Keep an extra sentence only if it states something not trivial and not visible in the nearby code. A sentence restating what sits a few lines away is a finding even when the rest of the comment is good — don't pass a comment whole; test it sentence by sentence.
- **Re-read after tightening.** After applying a tighter rewrite, re-read it fresh: if the shorter version lost meaning or reads as a fragment, keep the original. Over-tightening is a failure, not a win — "original is better than current" is a valid outcome of this review.
- **Stay 100% accurate.** A stale comment misleads — worse than none. Read the code the comment describes and verify every claim; a comment that's only true for one branch/path is a finding. When a code change lands next to an existing comment, re-verify that comment against the new code — one the change turned stale is a finding, even though you didn't write it.
- **Pre-existing comments: keep unless this change made them stale.** Incomplete is not stale — never extend, reword, or delete an accurate pre-existing comment. One that was wrong from the day it was written is a finding to report, not an edit to make; the user decides.
- **Volatile measured facts rot.** Counts, commit distances, current violation totals, and change history belong in the commit message, not a comment. And re-verify any measurable claim ("N pre-existing violations", "the hook already lints X") by running the measurement during the review — don't trust the text.
- **Never duplicate.** State a rationale once (in the docstring, the call site, or the test — not all three); point elsewhere if it's referenced again. Check across the whole diff AND against pre-existing comments — and also against adjacent runtime strings (a comment restating the `::warning::`/log message next to it), sibling field/type comments, and test-file headers repeating a source docstring.
- **Concise, plain, direct.** Omit needless words. Keep all technical terms, proper nouns, and code identifiers intact; drop decorative/emotional adjectives, emphasis caps, and flowery phrasing. If a comment needs paragraphs, the code needs a better name or a smaller function instead.
- **Simple English for new and edited comments.** If a `simple-english` skill (ASD-STE100) is available, apply it to comment text this change adds or edits, and to any rewrite this review proposes. It is NOT a license to rewrite pre-existing comments — a passing pre-existing comment stays untouched even if it breaks STE rules.
- **No PR/task narration in code** (e.g. "added for the empty-summary fix"). That belongs in the commit message — it rots as the codebase evolves.

Flag each offender with a tighter rewrite (or a delete).

## Check 2 — ID hygiene

- **KEEP in code** — a tracing or observability **trace ID** / **staging or production log** reference when the comment documents a **test fixture or reader payload** AND this repo has a fixture-contract requiring the real source be cited. Such a contract is **repo-specific** — check the repo's CLAUDE.md / AGENTS.md. Where such a contract applies, don't flag the citation; add it if missing. If the repo has no such contract, treat trace/log IDs like any other ID in the MOVE list below.
- **MOVE to the linked issue** — issue tracker IDs (`ABC-123`), session IDs, cloud log-stream names, PR numbers, customer names/identity, and similar explanatory references that are NOT fixture-source citations. These are reachable via git blame → PR → linked issue (per the "no tracker IDs in code/commits" rule).

For each ID flagged to move:
1. Find the linked issue (branch name, for example `feature/abc-123-…`, or the PR).
2. Check the issue's description and comments already contain it.
3. If missing, **draft** an issue-tracker comment to add it. If the repo documents tracker-formatting constraints, follow them. Show the draft; post only on explicit approval.

## Output

A checklist, most-actionable first:

| file:line | finding | action |
|-----------|---------|--------|
| `foo.py:42` | restates the next line | delete |
| `bar_test.py:88` | `ABC-123` reference | already in the issue → delete; else backfill then delete |
| `svc.py:15` | customer name in comment | already in issue → delete; else backfill then delete |
| `baz.py:10` | fixture missing its trace citation | add trace id per the repo's fixture contract |
| `qux.py:7` | wording nuance, branch already pushed | minor — no action |

End with one summary line of net comment volume: lines removed / added / net (e.g. "12 removed, 4 added, net −8") — comment growth should be visible at a glance.

If the branch is already pushed and a finding is only a minor wording nuance, report it as "minor — no action" rather than proposing an edit that needs its own commit.

Apply the comment edits on confirmation. Handle any tracker backfill as its own approved action. If there's no linked issue, just flag the IDs — don't invent a place to put them.
