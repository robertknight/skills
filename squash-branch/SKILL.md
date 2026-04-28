---
name: squash-branch
description: Squash all commits on the current git branch into a single commit. Soft-resets to the base branch, then re-commits the combined working tree with a single message.
disable-model-invocation: true
argument-hint: [commit-message]
---

Collapse every commit between the base branch and HEAD into a single commit on the current branch. This is a local history rewrite — the branch tip changes, but the working tree and index are preserved.

## Pre-flight checks (abort and report if any fail)

1. **Not on the base branch.** Determine the base (`dev` if it exists, otherwise `main`) and ensure the current branch (`git rev-parse --abbrev-ref HEAD`) is something else.
2. **Clean working tree.** `git status --porcelain` must show no modified or staged tracked files. Untracked files are fine. If anything is dirty, stop and tell the user to commit or stash first — otherwise the squash would slurp uncommitted work into the new commit.
3. **No mid-flight merge or rebase.** Refuse to run if `.git/MERGE_HEAD`, `.git/rebase-merge/`, or `.git/rebase-apply/` exists.

## Steps

1. Run `git rev-list --count <base>..HEAD`.
   - `0` → nothing to squash; report and stop.
   - `1` → already a single commit; report and stop unless the user supplied a new message via argument (in which case `git commit --amend -m <msg>` and stop).
2. Pick the commit message. The goal is a message that accurately describes the **end result** of the squash — i.e. the combined diff against the base — not just whatever the branch started out trying to do.
   - If the user supplied a `$ARGUMENTS` value, use it verbatim as the full message and skip the rest of this step.
   - Otherwise, start from the **oldest** commit's full message: `git log <base>..HEAD --reverse --format=%B -n 1`. The oldest subject usually frames the branch's overall purpose better than later fix-ups.
   - Then sanity-check it against the actual squashed change. Look at `git log <base>..HEAD --format=%B` (all messages, oldest first) and `git diff <base>...HEAD --stat` to see whether later commits expanded, narrowed, or redirected the work:
     - If later commits are pure fix-ups, formatting, review feedback, or test additions for the same feature, the oldest message is fine as-is.
     - If later commits added scope the original message doesn't mention (e.g. started as "fix X", later also refactored Y), edit the message so it covers the final scope.
     - If later commits reversed or replaced the original approach (e.g. first commit added a flag, a later commit removed it again), rewrite the message to describe what the final diff actually does — the abandoned approach shouldn't appear at all.
   - Keep the subject line concise and in the same style as the rest of the repo's history (`git log --oneline -n 20 <base>` for reference).
3. Check whether the branch has an upstream: `git rev-parse --abbrev-ref --symbolic-full-name @{u}` (suppress stderr). If it succeeds, remember to warn the user at the end that the next push will require `--force-with-lease`.
4. Run `git reset --soft <base>`. This is the only mutating command — destructive to branch history, but `ORIG_HEAD` and reflog preserve the previous tip for recovery.
5. Run `git commit` with the message from step 2, passed via heredoc:
   ```bash
   git commit -m "$(cat <<'EOF'
   <message body>
   EOF
   )"
   ```
   If the pre-commit hook fails, fix the issue and re-run `git commit` with the same heredoc (do **not** `--amend` — the squash commit was never created).
6. Verify with `git log <base>..HEAD --oneline` (must show exactly one commit) and `git status` (must be clean).
7. Report:
   - Old commit count → 1.
   - New commit SHA and subject.
   - Recovery hint: `git reset --hard ORIG_HEAD` restores the previous tip.
   - If the branch had an upstream (step 3): remind the user that pushing now needs `--force-with-lease`.

## Do NOT

- Push the result. The user pushes manually.
- Use `git rebase -i` — it requires interactive input and isn't supported here.
- Run on `main`, `dev`, or any other base branch.
- Auto-stash uncommitted changes — pre-flight check should bail instead.
