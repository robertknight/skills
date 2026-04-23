Analyze the changes on the current git branch compared to its base branch and provide a high-level summary that covers both *what* changed and *why*.

Steps:
1. Determine the base branch (typically `main` or `dev` — check which exists and is the likely merge target).
2. Run `git log <base>..HEAD --oneline` to see all commits on the branch.
3. Run `git diff <base>...HEAD --stat` to see which files changed and by how much.
4. Check for a PR description if one exists (`gh pr view --json body` — skip if no PR is open).
5. Read the key changed source files (prioritize new files and heavily modified files) to understand the purpose of the changes.
6. Produce a concise summary covering:
   - **Overall theme**: One-paragraph description of what this branch is about.
   - **Key changes**: Grouped by logical area (new modules, refactors, test changes, config changes, etc.) with bullet points for each. For each group, explain the likely rationale — why these changes were made, not just what they are.
   - **Net effect**: A one-sentence summary of the impact.

When explaining rationale, clearly distinguish between:
- **Stated rationale**: reasons evident from commit messages, PR description, code comments, or documentation changes. Present these as facts.
- **Inferred rationale**: your best guess at the motivation based on the nature of the changes. Mark these explicitly (e.g., "likely because…", "this suggests…", "presumably to…").

Keep the summary concise but thorough — focus on *what* and *why*, not line-by-line diffs.
