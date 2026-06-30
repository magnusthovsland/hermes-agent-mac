# GitHub fine-grained PATs, Git Data API fallback, and release verification

Use this reference when a user provides a GitHub fine-grained PAT, plain `git push` or `gh pr create` fails, but repository REST API access works.

## Permission model lessons

Fine-grained PAT permissions are endpoint-specific:

- `GET /repos/{owner}/{repo}` proves repository visibility only.
- **Contents: Read/Write** allows creating blobs/trees/commits and updating refs through the Git Data API.
- `POST /pulls` additionally requires **Pull requests: Read/Write**. If the token only has Pull requests: Read, PR creation can fail with:

```text
403 Resource not accessible by personal access token
```

Do not report this as “bad token” if repo and contents writes work. Report the exact missing permission.

## Safe token handling

- Do not write the token to config files unless explicitly asked.
- Use runtime environment variables or process-local variables.
- Avoid commands that echo token-bearing URLs.
- When diagnostics are needed, print HTTP status and resource names, never token contents.

## Git Data API fallback sequence

When `git push` over HTTPS fails but REST access works:

1. Read the target ref:
   - `GET /repos/{owner}/{repo}/git/ref/heads/{branch}`
2. Read its base commit and tree:
   - `GET /repos/{owner}/{repo}/git/commits/{sha}`
3. For each changed file, create a blob:
   - `POST /repos/{owner}/{repo}/git/blobs`
4. Create a tree with `base_tree` and changed file blobs:
   - `POST /repos/{owner}/{repo}/git/trees`
5. Create a commit with parent = base branch SHA:
   - `POST /repos/{owner}/{repo}/git/commits`
6. Update or create the branch ref:
   - direct branch update: `PATCH /git/refs/heads/{branch}` with `force: false`
   - new PR branch: `POST /git/refs` with `refs/heads/<branch>`
7. Read back refs after the write and report before/after SHAs.

## Direct QA + PR-to-prod pattern

If the user asks “take code directly to QA and create PR to prod”:

1. Identify actual deployment/prod branch names from remotes and workflow files. Do not assume `qa` deploys just because the branch is named `qa`.
2. Create one commit on top of the QA branch and update `qa` directly.
3. Create a separate branch from prod/main with the same content for review.
4. Open PR to prod/main if the token has Pull requests write permission.
5. If PR creation is blocked, provide the compare URL:

```text
https://github.com/<owner>/<repo>/compare/<base>...<head>?expand=1
```

## Release/deploy verification

Do not answer “yes, released” from branch update alone.

Verify at least one of:

- workflow file trigger and recent Actions run for the relevant branch/commit,
- deployment provider status if available,
- live QA/prod URL behavior with cache headers and actual HTML/source check.

For SEO/meta fixes, verify via HTTP source, not only browser DOM:

```text
<title>
<meta name="description">
<meta property="og:title">
<meta property="og:description">
```

A branch update can be correct while no deployment job runs if the workflow is configured for a different branch or only manual dispatch.
