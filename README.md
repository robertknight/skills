# Claude Code skills

This directory contains shared skill definitions for use with Claude Code or
other agents.

## Installation

Clone the repository to `~/.claude/skills`.

## Available skills

- **[audit](audit/SKILL.md)** — Quick triage pass for obvious malicious or untrustworthy behavior in a project, so the user can decide whether it's safe to run, build, or install.
- **[explain-branch](explain-branch/SKILL.md)** — Summarize the changes on the current branch against its base, covering both *what* changed and *why*.
- **[read-branch](read-branch/SKILL.md)** — Load the full diff of the current branch against its base into context for follow-up prompts, with a brief grouped summary.
- **[review-branch](review-branch/SKILL.md)** — Review the diff of the current branch against the base branch, critique the changes, and fix any issues found.
- **[squash-branch](squash-branch/SKILL.md)** — Squash every commit on the current branch into a single commit via a soft reset to the base.
