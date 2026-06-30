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

### Fine-grained PAT and REST fallback
When a user provides or points to a GitHub fine-grained PAT, verify exact capability before assuming `git push`/PR creation will work:
```bash
curl -sS -H "Authorization: Bearer $TOKEN" \
  -H 'Accept: application/vnd.github+json' \
  https://api.github.com/repos/OWNER/REPO
```
If REST repo access and **Contents: read/write** work but HTTPS git transport is blocked by credential helper/auth format, use the GitHub Git Data API as a safe fallback for branch updates:
1. `GET /git/ref/heads/<base>` and `GET /git/commits/<sha>`.
2. Create blobs for changed files with `POST /git/blobs`.
3. Create a tree with `POST /git/trees` using `base_tree`.
4. Create a commit with parent `<base_sha>` via `POST /git/commits`.
5. Update or create refs with `PATCH /git/refs/heads/<branch>` / `POST /git/refs`.
6. Read back refs before reporting success.

Important permission split for fine-grained PATs:
- Updating code/branches needs repository **Contents: read/write**.
- Creating a PR via `POST /pulls` separately needs **Pull requests: read/write**. If only Pull requests: read is granted, create/update the branch and return the compare URL for manual PR creation.

Monitor CI:
### QA-direct + prod-PR release pattern
When the user asks to “put it in QA” while also creating a PR for production review, treat these as two separate tracks unless they explicitly say otherwise:
1. Apply/push the tested change directly to the QA branch/environment for validation.
2. Create a separate branch from the production base (usually `main`) with the same change.
3. Open the PR against the production branch, not QA.
4. Verify the QA deployment/live URL independently before telling the user it is testable.

Do not assume a branch named `qa` triggers deployment; inspect `.github/workflows/*`, deployment provider state, or the live QA URL. A workflow file named “deploy_to_qa” may still trigger on `main` or deploy only a subcomponent.

### PAT scope and GitHub API fallback pitfalls
Fine-grained GitHub PATs can have `contents:write` but only `pull_requests:read`. In that state, branch/commit writes may succeed while `POST /pulls` fails with `403 Resource not accessible by personal access token`. Ask for/update `Pull requests: Read and write` before retrying PR creation.

If `git push` over HTTPS rejects a token but REST API access works, use GitHub Git Data API as a fallback for branch writes:
- read base ref (`GET /git/ref/heads/<branch>`),
- create blobs for changed files,
- create tree with `base_tree`,
- create commit with parent SHA,
- update/create ref.
Still read back the branch/PR after the API write before reporting success.

Monitor CI:
```bash
git format-patch -1 HEAD --stdout > /tmp/<descriptive-branch>.patch
```
Report the branch name, commit SHA, verification output, auth blocker, and patch path. Treat this as a fallback deliverable, not as a substitute when authenticated push is available.

### Fine-grained PAT and Git Data API fallbacks

Fine-grained GitHub PAT permissions are endpoint-specific. A token with **Contents: Read/Write** may create commits/refs via the Git Data API but still fail `POST /pulls` unless **Pull requests: Read/Write** is granted. Conversely, REST repo access can work while `git push` over HTTPS fails because the Git credential path is not using the same token.

When a user provides a PAT and `git push`/`gh pr create` fails:
1. Verify repo access with `GET /repos/{owner}/{repo}` without printing the token.
2. If Contents write works but git transport fails, use Git Data API to create blobs, a tree, a commit, and update/create refs; read back the target refs afterward.
3. If PR creation returns `403 Resource not accessible by personal access token`, report that **Pull requests: Read/Write** is needed and provide the compare URL (`/compare/base...head?expand=1`).
4. Never conclude a release happened from a successful commit alone; inspect workflows and live deployment output.

Operational details and example sequence: `references/github-fine-grained-pat-git-data-api.md`.

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
