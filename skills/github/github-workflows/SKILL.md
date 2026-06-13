---
name: github-workflows
description: "Class-level umbrella for GitHub work: auth, repositories, issues, pull requests, CI monitoring, and code review using gh, git, and REST fallbacks."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [github, git, gh, pull-requests, issues, code-review, ci, releases]
    related_skills: [requesting-code-review, autonomous-coding-agent-clis]
---

# GitHub Workflows

## Overview
Use this umbrella whenever work crosses the Git/GitHub boundary: authenticating, cloning/forking/creating repos, managing issues, creating/reviewing PRs, monitoring CI, releases, repository settings, or branch protection.

Default hierarchy:
1. Use `gh` when installed and authenticated.
2. Use plain `git` for local repository operations.
3. Use GitHub REST via `curl` with `GITHUB_TOKEN` when `gh` is unavailable.
4. Read back every external side effect before reporting success.

## Initial Discovery
Run this before GitHub operations:
```bash
git --version
gh --version 2>/dev/null || echo "gh not installed"
gh auth status 2>/dev/null || echo "gh not authenticated"
git remote -v 2>/dev/null || true
```
Extract owner/repo from the remote when needed:
```bash
REMOTE_URL=$(git remote get-url origin)
OWNER_REPO=$(echo "$REMOTE_URL" | sed -E 's|.*github\.com[:/]||; s|\.git$||')
OWNER=$(echo "$OWNER_REPO" | cut -d/ -f1)
REPO=$(echo "$OWNER_REPO" | cut -d/ -f2)
```

## Authentication
Use `gh auth status` as the preferred readiness check. If not available:
- HTTPS PAT: configure `GITHUB_TOKEN` or git credential helper.
- SSH: verify keys with `ssh -T git@github.com`.
- Headless `gh`: use token-based login (`gh auth login --with-token`) only when the user provides/authorizes a token.

Never print secrets. Report tokens as SET/not set, and avoid copying credential files into artifacts.

## Repository Management
Use for clone/create/fork/remotes/settings/releases/secrets.
```bash
gh repo clone owner/repo
# or
git clone https://github.com/owner/repo.git

gh repo create owner/name --private --source=. --remote=origin --push
gh repo fork owner/repo --clone
```
For protected settings, releases, or secrets, prefer `gh`/REST and read back the setting/release list after changes.

## Issues
Use for searching, creating, triaging, labeling, assigning, and closing issues.
```bash
gh issue list --state open --label bug
gh issue view <number> --comments
gh issue create --title "..." --body-file body.md --label bug
gh issue edit <number> --add-label triaged --assign @me
```
When creating issues, use structured reproduction steps, expected/actual behavior, environment, and acceptance criteria. Use templates for bug/feature reports when present.

## Pull Request Lifecycle
Use for branch/commit/push/PR/CI/merge.
```bash
git checkout -b fix/descriptive-name
git status --short
git diff --stat
git add <files>
git commit -m "fix: concise subject"
git push -u origin HEAD
gh pr create --fill
```
Monitor CI:
```bash
gh pr checks <number> --watch
# or
gh run list --branch <branch> --limit 10
```
Do not merge until requested and CI/review requirements are understood.

## Code Review
Use for pre-push local review or PR review.
```bash
git diff --staged
git diff origin/main...HEAD
gh pr diff <number>
gh pr view <number> --comments --json files,commits,statusCheckRollup
```
Review by severity:
- Critical: correctness/security/data loss/build break.
- Warnings: maintainability, edge cases, flaky tests.
- Suggestions: style and future improvements.
- Looks good: explicitly state what was checked.

When posting review comments, quote exact file/line and verify the comment appears.

## Verification Checklist
- [ ] Auth method is known and secrets are not exposed.
- [ ] Repo/owner/branch/PR/issue identifiers are read from live state.
- [ ] Local diff/status is inspected before commit or review.
- [ ] External writes are read back via `gh`, REST, or `git`.
- [ ] Final summary includes URLs/numbers only after verification.
