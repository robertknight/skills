---
name: review-branch
description: Review the diff of the current branch against the base branch, critique the changes, then fix any issues found.
disable-model-invocation: true
argument-hint: [base-branch]
---

Review the diff of the current branch against the base branch and critique the code changes. Don't make any edits yet, just provide a review with issues categorized as major, minor, or nitpick.

## Determining the base branch

If an argument is provided (`$ARGUMENTS`), use that as the base branch. Otherwise, detect the default branch from the remote by running `git symbolic-ref refs/remotes/origin/HEAD` and extracting the branch name. If that fails, fall back to whichever of `main`, `master`, or `dev` exists locally (check in that order with `git rev-parse --verify`).

## Review guidelines

- Code should be written under the assumption that values conform to their type annotations. Flag `isinstance` checks or other runtime type guards that are redundant given the declared types. If values don't actually conform to the annotations, the annotations are wrong — that's the bug to fix, not a reason to add defensive checks.

## After the review

Ask whether to fix the issues. If yes, fix them, run typecheck/lint/tests to verify, and commit the fixes.
