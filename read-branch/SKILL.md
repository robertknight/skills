---
name: read-branch
description: Read the full diff of the current branch against its base so the assistant has context for follow-up prompts. Produces a brief grouped summary, not a critique.
disable-model-invocation: true
argument-hint: [base-branch]
---

Load the diff on the current git branch into context and produce a short summary of what changed, grouped by area. The point is shared context for the user's follow-up prompts — not a review, critique, or rationale analysis.

Steps:
1. Determine the base branch. Use the argument if provided; otherwise pick whichever of `dev` or `main` exists and is the likely merge target (check `gitStatus` context first if available).
2. In parallel, run:
   - `git diff <base>...HEAD --stat` — file-level overview
   - `git log <base>..HEAD --oneline` — commits on the branch
3. Run `git diff <base>...HEAD` to load the full diff into context. If the diff is very large (e.g. >1500 lines from the stat), fall back to reading the most-changed files individually with the Read tool instead of dumping everything.
4. Produce a concise summary:
   - Group changes by logical area (e.g. directory, package, layer — editor / styles / sidebar / tests / config).
   - Under each group, bullet the concrete changes (new files, new functions, attribute additions, behavioral tweaks). Reference paths so the user can navigate.
   - Keep it tight — a few bullets per group, not a paragraph each.
5. End with a one-line "Ready for your next prompt." style sign-off.

Do NOT:
- Critique the code, suggest fixes, or evaluate quality.
- Speculate at length about rationale (a brief "why" is fine when the diff makes it obvious; otherwise skip it).
- Make any edits. This is a read-only context-loading skill.
