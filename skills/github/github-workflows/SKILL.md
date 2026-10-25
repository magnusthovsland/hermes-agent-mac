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

Validate a newly supplied token directly against the intended repository without first replacing global CLI authentication; stale keyring credentials do not establish that the new token fails. If the owner/repo is uncertain, discover accessible repositories and then read the exact target. Treat screenshots and API `permissions` booleans as evidence of configured access/account role, not proof of all fine-grained token write scopes. Report only the capability actually exercised; a successful repository read does not verify branch writes or PR creation. Do not make external test writes solely to prove permissions. Keep temporary credentials separate from standing credentials and recommend revocation after the authorized work.

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
For this user's Wright-Web work, write tested changes directly to `qa`; the release PR is **from `qa` to `main`**. Do not silently substitute a feature branch, draft PR into QA, or Vercel Preview for a requested QA delivery. Production release still requires its own authorization.

1. Read the current remote QA head, deployment mapping and effective branch rules before choosing the delivery path.
2. Apply the scoped tested change directly to QA when permitted, without force. If rules contradict the requested direct-write workflow, report the exact conflict before creating an alternative PR or preview deployment; obtain agreement on the alternative or an authorized rules change.
3. Verify the updated QA ref and the actual QA-site deployment, not only a feature-branch build or Vercel Preview status.
4. When production release is requested, open the release PR with head `qa` and base `main`; do not create a separate production feature branch by default.

For Wright-Web, verify frontend QA at `https://qa.wright.no/` after updating `qa`; Vercel's Git integration can deploy the frontend independently of GitHub Actions. The Vercel bot's PR comment identifies project `teoriportalqa/wright-web` and preview status even when a supplied PAT cannot read Checks/Deployments (403). Verify the actual QA behavior, not merely the preview status. For package ordering, compare rendered package slugs mapped to Sanity product IDs against `https://qa.ovio.no/api/public/office-packages/all?office_id=…` using `X-Tenant-Id: 1` and `X-System: wright-web-frontend`; map every CMS slug rather than assuming one document per externalId.

Do not assume a branch named `qa` exists or triggers deployment; verify the exact remote ref, inspect `.github/workflows/*`, and check deployment-provider configuration. A workflow file named “deploy_to_qa” may still trigger on `main` or deploy only a subcomponent.

### Branch rules and reviewer identity preflight

- Query `GET /repos/{owner}/{repo}/rules/branches/{branch}` for effective rules, including repository/organization rulesets, before interpreting `protected: true`. Inspect `type: pull_request` and its approval parameters; the protected flag alone does not prove that direct writes are forbidden.
- Read classic protection with `GET /repos/{owner}/{repo}/branches/{branch}/protection` when accessible. A 403 on this endpoint is not proof of a rule or of all rule-read access being unavailable; the effective-rules endpoint may still succeed. Report what was actually verified.
- Do not route through Git Data API, disable protection, or force-push to evade a known rule. If the intended direct-QA workflow conflicts with active rules, request a scoped correction while preserving production review requirements; do not assume the rules are permanently configured this way.
- Before asking the user to approve a PR, compare `GET /user` with the PR's `user.login`. A PR created with the user's token is authored by that user, who cannot provide its required self-approval. Explain that another eligible reviewer is needed rather than repeatedly asking the author to approve.
- Read back `merged` and the destination branch ref after a merge attempt. A non-null `merge_commit_sha` on an open PR can describe a test merge and does not prove deployment or a completed merge.

### Missing branch versus surviving deployment

1. Confirm the exact owner/repository and case-sensitive branch name. If a branch lookup returns 404, first establish repository access; then check `GET /repos/{owner}/{repo}/git/ref/heads/{branch}`, paginate the branch list, and independently run authenticated `git ls-remote --heads <remote>`. On a disputed result, repeat uncached reads rather than relying on the earlier list. Keep credentials out of command arguments and output.
2. Distinguish a Git ref, a GitHub deployment environment, and a Vercel deployment. A deployment marked Ready/Active or labeled with a branch proves deployment history, not that the source ref currently exists. Read screenshots according to the page type; an environment's deployment list is not the repository branch selector.
3. Report the observed scope precisely: 'the exact ref is absent from the authenticated API and remote Git listing.' To establish deletion timing, read the release PR's `GET /repos/{owner}/{repo}/issues/{number}/timeline` for `head_ref_deleted`/`head_ref_restored` and corroborate with repository `events` entries of type `DeleteEvent` whose payload names the exact branch. Paginate when needed; missing events do not prove no deletion because event history is bounded. Convert UTC timestamps to the user's timezone with a tool. Describe the actor as the account recorded by GitHub; this alone does not distinguish a manual click from automatic branch deletion after merge. A surviving site does not contradict an absent branch.
4. Before proposing restoration, match the active deployment's full source SHA with repository commit/PR metadata. Do not silently recreate a missing QA branch from today's production head; the last deployed QA source may differ. Inspect workflows for all branch-creation side effects, including CMS deployment as well as frontend deployment.
5. Obtain explicit approval for the restoration baseline and resulting deploys, then read back the restored ref and verify the actual deployment separately. If the user only asked to inspect an image or verify branch existence, do not restore or resume implementation.

### Disputed branch equality and safe QA synchronization

1. Resolve both remote heads afresh through GitHub API and `git ls-remote --heads https://github.com/OWNER/REPO.git refs/heads/qa refs/heads/main`. Verify repository `full_name`, fork status, and case-sensitive branch names. Explain that `ls-remote` queries the server, not local tracking refs; provide the exact reproducible command when a developer disputes the finding.
2. Pin the returned SHAs and compare `GET /repos/{owner}/{repo}/compare/{qa_sha}...{main_sha}`. State the direction explicitly: `ahead_by` counts commits main has beyond QA in this request. Compare commit tree SHAs as well: different commit IDs can have identical source trees after a merge. For diverged histories, use the trees or a fetched two-endpoint `git diff <qa_sha> <main_sha>` to establish tip-content differences; a merge-base comparison alone is insufficient. Do not treat a truncated API file list as the full changed-file count.
3. Compare the pre-merge QA head from the release PR separately from the currently restored QA head. This distinguishes 'QA matched main at release' from 'the current QA ref matches main.' A commit's timestamp dates the commit, not creation of the restored branch.
4. Report a short evidence block: exact repository, remote SHAs, tree equality, direction/count of history differences, and representative file differences. Provide the GitHub compare page when useful. Distinguish authenticated API evidence from an actually inspected web page; do not claim visual confirmation without reading it. Report observed divergence without guessing whether the developer intended it.
5. When synchronization is authorized and QA is an ancestor of main, prefer a non-force fast-forward over deleting and recreating QA. Inspect branch protection and deployment side effects first; if rules require a PR but the user requested direct QA writes, explain the mismatch and agree on the path rather than silently creating a PR. Do not bypass protection. Re-read both heads immediately before writing and stop if they changed. Verify the resulting ref and deployment separately. For a capability-only question or 'do not do it', explain this route without making changes.

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
