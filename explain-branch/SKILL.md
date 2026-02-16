Analyze the changes on the current git branch compared to its base branch and provide a high-level summary.

Steps:
1. Determine the base branch (typically `main` or `dev` — check which exists and is the likely merge target).
2. Run `git log <base>..HEAD --oneline` to see all commits on the branch.
3. Run `git diff <base>...HEAD --stat` to see which files changed and by how much.
4. Read the key changed source files (prioritize new files and heavily modified files) to understand the purpose of the changes.
5. Produce a concise summary covering:
   - **Overall theme**: One-paragraph description of what this branch is about.
   - **Key changes**: Grouped by logical area (new modules, refactors, test changes, config changes, etc.) with bullet points for each.
   - **Net effect**: A one-sentence summary of the impact.

Keep the summary concise but thorough — focus on *what* and *why*, not line-by-line diffs.
