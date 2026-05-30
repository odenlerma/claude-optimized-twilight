# Project setup and vibe-coding workflow rules

Reusable across any software project. Applies when starting or running an AI-assisted project.
Load via `/vibe-setup-rules`.

## 1. Context file first
Create a `CLAUDE.md` (or agent context file) with stack, conventions, key commands, and architecture before generating much code. Good context = AI output that fits your project instead of fighting it. Use built-in `/init` to bootstrap it.

## 2. Plan before you generate
Write a short spec — goal, acceptance criteria, constraints — before asking AI to implement. Generating against a plan beats generating then patching. Vague prompt → plausible wrong code.

## 3. `.gitignore` + `.env.example` on day one
Set up secret-ignoring and a blank env template before the first commit. Day-one hygiene means secrets never get committed in the first place. Retrofitting after a leak means rotating keys.

## 4. Checkpoint discipline
Small commits on a branch, especially around large AI changes. Frequent checkpoints make any bad generation a one-command rollback. Don't let unreviewed AI changes pile up uncommitted.

## 5. Pin the toolchain
Lockfiles for dependencies, a version manager (mise/asdf/nvm/uv) for language/tool versions. "Works on my machine" dies when versions drift. Reproducible environment from the start.

## 6. Wire automation early
Formatter, linter, type-check, and pre-commit hooks before code piles up. Automated guardrails on day one cost minutes; retrofitting them onto a messy codebase costs days. Let tools enforce, not arguments.

## 7. Define "done"
A change is done when tests pass, lint is clean, types check, no secrets, and a human read the diff. Write the checklist down. "It runs" is not done.

## 8. Keep dependencies minimal at the start
Add a dependency only when you need it, and vet each one (maintained, popular, sane license). Every dependency is attack surface and maintenance debt. AI loves to pull in packages — push back.

## 9. README from the start
Document how to install, run, test, and configure from the first commit. A project nobody (including future-you) can start is broken. Keep it current as setup changes.

## 10. Review AI output — you own the code
Read every diff before merging; never ship code you don't understand. AI accelerates typing, not accountability. The human signs off on correctness, security, and intent.
