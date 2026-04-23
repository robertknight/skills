Quickly audit the current project for obvious malicious or untrustworthy behavior so the user can decide whether it is safe to run, build, or install. This is a triage pass, not a full security review — aim for ~a dozen targeted checks, not an exhaustive audit.

Goal: surface anything that looks suspicious. Do not try to prove the code is secure. If nothing suspicious turns up after the checks below, say so and stop.

Steps:

1. **Identify the project.** Read the root manifest (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `setup.py`, etc.), `README.md`, and `LICENSE`. Note the declared purpose, language, and whether it matches what a casual user would expect (e.g., a "diff viewer" shouldn't need Bitcoin wallets as dependencies).

2. **List the top-level layout.** `ls -la` the repo root. Flag unusual hidden files, binaries checked into source, or suspiciously named directories.

3. **Dependency review.** Scan the dependency list for anything unexpected given the project's stated purpose — typosquatting-style names, obscure/unmaintained packages, crypto/networking libs in tools that shouldn't need them, packages pulled from non-standard registries or git URLs.

4. **Build-time code execution.** Check for code that runs during install/build:
   - Rust: `build.rs`
   - Node: `scripts` in `package.json` (preinstall, postinstall, prepare)
   - Python: `setup.py` (arbitrary code), non-PEP517 builds
   - Go: `//go:generate` directives, cgo
   - Any Makefile / shell script pulled in by the build

5. **Network activity.** Grep for HTTP clients and URLs (`reqwest`, `ureq`, `http`, `fetch`, `axios`, `requests`, `urllib`, `curl`, `wget`, hardcoded `https?://`). For each hit, check what URL is contacted and what data is sent. Flag anything that looks like telemetry, exfiltration, or contacts a non-obvious third-party host.

6. **Subprocess execution.** Grep for process spawning (`Command::new`, `exec`, `spawn`, `subprocess`, `os/exec`, `child_process`, `system(`, `popen`, `eval(`, `Function(`). For each, check: is the argv hardcoded, or built from user/network input? Shell interpolation is a red flag.

7. **File/system access.** Look for writes outside the project directory, especially to `~/.ssh`, `~/.aws`, browser profile dirs, keychains, or crontabs. Grep for those paths.

8. **Obfuscation.** Look for base64 blobs, hex-encoded strings, `eval`/`exec` on decoded data, minified code in a source repo, or binaries in unexpected places.

9. **Unsafe / FFI.** For memory-unsafe languages: grep for `unsafe` blocks, `dlopen`, `LoadLibrary`, inline assembly. A handful in test code is normal; large amounts in core logic warrants a closer look.

10. **CI / workflows.** Skim `.github/workflows/`, `.gitlab-ci.yml`, etc. Flag workflows that run on `pull_request_target`, use untrusted input in shell contexts, or exfiltrate secrets.

11. **Git hooks & tool configs.** Check `.git/hooks/` (if committed via a hooks dir), `.husky/`, `.pre-commit-config.yaml`, `.vscode/tasks.json`, `.claude/`, `.cursor/`, `.devcontainer/` for auto-run commands.

For each finding, classify as:
- **Expected** — matches the project's stated purpose, hardcoded args, reputable dep.
- **Noteworthy** — worth mentioning but not alarming (e.g., a single update-check call, OS integration subprocess).
- **Suspicious** — unexplained network call, obfuscated code, install-time script doing unusual things, dependency mismatch with stated purpose.

Output format:

- Start with a one-line verdict: **safe to use**, **looks fine with caveats**, or **do not run — suspicious behavior found**.
- Then a short "What it is" line (name, language, purpose, source).
- Then a compact list of findings grouped by category (Network, Subprocess, Build scripts, Unsafe, etc.). Include file:line citations. Skip categories with nothing to report.
- End with anything the user should confirm themselves (e.g., "I didn't verify the signatures on the release binaries" or "check the publisher's reputation if installing from crates.io").

Keep it tight. The user is deciding whether to trust the code — not reading a compliance report.
