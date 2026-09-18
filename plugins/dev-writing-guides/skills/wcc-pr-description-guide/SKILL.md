---
name: wcc-pr-description-guide
description: Guide and review checklist for a PR title & description - reviewer-lens discipline, reviewer comprehension, sync with the actual code, and format/ID hygiene. Use when drafting a PR description, reviewing one, or asked to follow the PR description guide. Args - [pr-number | pr-url | draft-file] (default - the current branch's open PR).
---

Review **the PR title and description**, checking every claim against the actual branch state. The description is the reviewer's sync channel — same minimal/accurate/why-not-what discipline as code comments, plus the requirement that it stays in sync with the evolving code.

## Scope

- No arg → the current branch's PR: `gh pr view --json number,title,body,baseRefName`.
- Arg is a PR number or URL → that PR: `gh pr view <arg> --json number,title,body,baseRefName`.
- Arg is a file path → review that local draft (pre-PR).

For the sync check, load the code the description claims to describe:

- **Explicit PR arg** → that PR's own diff and commits: `gh pr diff <number>` and `gh pr view <number> --json commits`. Never the local checkout — it may be on a different branch, and the PR's base is `baseRefName`, not necessarily `main`.
- **No arg / local draft** → the current branch against the PR's base (default `origin/main` when there's no PR yet): `git diff $(git merge-base origin/<base> HEAD)...HEAD` and `git log --oneline` over the same range — this covers unpushed commits.

Also load the repo's `.github/PULL_REQUEST_TEMPLATE.md` if present.

## Check 1 — reviewer-lens content

Keep only what the reviewer needs; everything else is a finding.

- **Title readable outside the session.** No jargon or invented terms in the title (`no-op'ing`); prefer stating what the change does over labeling the defect ("make agent lint step actually lint changed files", not "repair silently no-op'ing lint step"). After a title rename, check the branch's commit subject: squash-merge uses the PR title, but a merge commit keeps commit subjects — flag a subject still carrying the old wording.
- **Diff-derivable → cut.** If reading the code diff shows it, don't write it. Implementation details are for the reviewer to read in code — inline them only when the PR is very small. A rationale that already exists as a code comment in the diff is diff-derivable — cut it from the description.
- **No invented wording.** Every term that names a mechanism must already exist in the codebase or repo docs — grep for what the code calls it (a log message, identifier, or comment usually supplies the established term). If none exists, describe the mechanism in plain words instead of naming it. No idiom, no analogy, no new coinage.
- **Reviewer comprehension.** The reviewer may not know the subsystem internals. Each bullet must carry its own context: if it relies on a relationship (task X writes cache Y), state the relationship in the bullet or cut the bullet. Every noun has a referent on first use; no bare "the task" / "the marker" / "the reader". Each sentence must survive one read.
- **Decisions only for cases that can happen.** A "non-obvious decision" bullet must describe a case that can actually occur. A decision written for an impossible case is a cut (or, if the impossibility itself matters to the reviewer, state plainly that the case cannot occur and why).
- **Shorten by deleting, not compressing.** Meet the length gate by cutting whole items, never by squeezing a sentence into a fragment. A shorter but unreadable sentence is a new finding, not a fix.
- **Length gate, scaled to the diff.** After the cuts above, re-read as the reviewer: for a small diff (under ~100 lines) the body should fit on one screen (~350 words). Longer than that → cut again or move detail to the tracker issue. A description that survives every other check can still fail this one.
- **Verification: default omit.** Basic verification (tests pass, lint, build) is not written. Keep only tests/evidence the reviewer specifically must know about, in a short section (a few lines, not a walkthrough). Docs-only PR → one line ("Docs-only change; no code, tests, or build touched.").
- **Non-obvious decisions**: decision rationale + spots needing extra review attention only. Never restate what the change does and why.
- **Background = objective facts.** The author's personal preferences must not be framed as background rationale.
- What belongs: context, why, scope of the change, and gotchas — the things the diff can't show.

## Check 2 — sync with code

Verify every remaining claim against the current branch state — a stale description misleads like a stale comment.

- **Claims match code**: behavior described, file paths, names, blocked/allowed lists, defaults — all as the code now is, not as it was when the PR opened.
- **Naming drift**: terms in the description must match the identifiers actually in the diff (a rename in code strands the description).
- **Commit coverage**: the description covers what the branch now contains, and nothing it no longer contains (later fixes, reverts, review-response commits included).

## Check 3 — format & ID hygiene

- **Repo template verbatim.** Copy the template's headers as-is, including its heading style; no invented sections. No template in the repo → these sections, in this order: `## Background`, `## Change`, `## Non-obvious decisions`.
- **Title: no issue tracker ID** (`ABC-123`). **Description: tracker ID is fine** (e.g. `Fixes ABC-123`).
- **MOVE to the tracker issue**: session IDs, log-stream names, customer identity, milestone labels, internal process stage names. Backfill before deleting — the information must land in the issue first:
  1. Find the linked issue (branch name, for example `feature/abc-123-…`, or the PR).
  2. Check whether the issue's description or comments already contain it.
  3. If missing, draft the issue comment, show it in chat, post only on explicit approval.
  4. Only then remove it from the PR text. No linked issue → just flag the ID; don't invent a place to put it.
- **No hard line-wrapping** in the body (GitHub renders markdown).
- **Style**: plain, direct, objective. Technical terms, proper nouns, and code intact; drop decorative adjectives and flowery/formal phrasing. Em-dash: none anywhere — title, prose, bullets, tables. Replace with a colon, comma, period, or parentheses. Prefer words over arrow symbols (`→`) in prose.
- **Simple English by default.** If a `simple-english` skill (ASD-STE100) is available, apply it to all text you write or rewrite here: short sentences, one idea per sentence, no filler. Load that skill when drafting; don't wait to be asked.

## Output

A checklist, most-actionable first:

| location | finding | action |
|----------|---------|--------|
| title | carries `ABC-123` | move to description (`Fixes ABC-123`) |
| Non-obvious decisions ¶2 | restates what the change does | delete |
| How ¶1 | says `cache_dir`, code now uses `CACHE_DIR` | re-sync wording |
| Non-obvious decisions ¶3 | "all tests pass" | delete (basic verification) |

Then show the full rewritten title + description in chat. Update the PR only on explicit approval, via `gh pr edit <n> --title ... --body-file <tmpfile-with-PR-number-in-name>`. For a local draft, just apply the edits to the file on confirmation.
