# Preparing a migration-preserving feature reversal

Use when a feature must be removed while its database migration has already run in a shared environment. This recipe prepares a reviewable reversal; it does not establish that subsequent merge/deploy steps have happened.

## 1. Pin scope and deployment consequences

Read the original PR's current state, head and base, fetch explicit branch refs, and inspect workflow triggers. Define the exact migration paths to retain. Compare other active branches before assigning new migration versions; report availability as observed in remote source, not proof about unpublished developer work or database history.

Prefer a scoped change with an explicit schema dependency over silently mixing unrelated feature code. If the user chooses merge-then-revert, preserve that choice and agree whether the intermediate deployment is held or knowingly allowed; do not alter workflow permissions, approvals or runs yourself. This recipe retains migrations only when that is the agreed scope. For a requested complete source reversal, the retained set may instead be empty, but applied schema/history cleanup remains a separate, explicitly authorized database operation whose safety is not established by this recipe.

## 2. Build the reversal from the feature head

Create a clean worktree on a new branch from the pinned feature SHA. Obtain the common baseline with `git merge-base <target-ref> <feature-ref>` and enumerate `git diff --name-status <baseline> <feature-ref>`.

For each changed path not in the retained-migration set, restore the baseline version using `git restore --source=<baseline> --staged --worktree -- <path>`. Handle added, removed and renamed paths explicitly; verify the resulting tree rather than assuming a filename loop reversed every operation. Do not reset the whole tree to today's target branch: that can mix unrelated newer target changes into the reversal diff.

Check:
- `git diff --cached --check` passes.
- `git write-tree`, compared with the baseline, differs only by the exact retained migration set.
- Each retained migration's bytes match the original feature SHA; compute its checksum when matching applied Flyway evidence.
- The reversal diff against the feature removes only the agreed feature changes.

## 3. Exercise the candidate

Run the repository build and tests against an isolated database with external service boundaries mocked or blocked. Aggregate actual JUnit XML counts, including failures, errors and skipped tests; preserve logs and stop only the isolated database after testing.

After committing, use `git merge-tree --write-tree <current-target> <candidate>` to inspect the prospective result without modifying target refs. Check the command succeeds, then compare that tree with the pinned current target: only the intended retained schema additions should remain. This protects newer target changes that were absent from the feature branch. Tree equivalence is source evidence, not an executed post-merge deployment or seeded-database upgrade test.

## 4. Publish a clearly gated draft

Push only the new branch, then open a draft against the original feature branch so the subtraction is visible. Read back base, head, draft state and changed files; verify retained migrations are not modified by the reversal diff.

The PR body must say:
- Do not merge into the temporary feature base.
- Which migration files remain and why.
- Which code and test changes are removed.
- The original and target SHAs used for verification.
- Actual test results and verification limits.
- Whether branch-filtered CI runs for this temporary base. A main-only PR workflow may not run yet; say so instead of treating missing checks as green.
- Agent implementation disclosure and the user's required commit identity.

## 5. Gate the later transition

After the original PR is actually merged, re-read the resulting target commit and parents, the dependent PR's current base and files, and triggered runs. GitHub may have automatically retargeted the PR. After squash merge, the old common ancestor can make its diff show only the retained migration instead of the reversal; an apparently successful retarget is not proof of the right changes.

For an authorized non-force adaptation of an already-published reversal:
1. Pin the existing reversal head and its exact reversal-path list before modifying it. Fetch the current target explicitly and check the local worktree and remote feature ref are unchanged.
2. Run `git merge --no-ff --no-commit <target-sha>`. A clean merge can silently reintroduce the feature from its squash commit, so inspect the staged tree even when Git reports no conflicts.
3. Restore only the reviewed reversal paths from the pinned pre-merge reversal head with `git restore --source=<reversal-head> --staged --worktree -- <paths>`. Do not restore unrelated paths from the old tree; that would discard newer target work. Review intervening edits to overlapping files rather than overwriting them blindly.
4. Use `git write-tree` and tree comparisons to prove the result equals the intended pre-feature code plus the explicitly retained migrations and any unrelated target changes. Check the final subtraction against the actual merged target, not only against the original feature branch.
5. Rerun a clean build and the relevant full test suite; when retaining an applied migration, reuse an isolated seeded database to exercise validation as well as fresh-database coverage. Commit the merge adaptation normally and push without force.
6. Read back the PR base, exact head and complete changed-file list, update stale temporary-base instructions, and verify CI for that new head before marking ready. Keep review and merge approval separate from passing tests.

Keep the original feature branch until the dependent PR has been handled. Explain that Git remembers a reverted merge: later feature restoration may require a revert-of-revert or fresh corrected commits, not merely merging the old branch again. Verify the final deployed SHA separately from both merge operations.
