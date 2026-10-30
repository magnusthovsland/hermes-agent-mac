# Dependency maintenance alongside feature delivery

## Execution order

1. Pin the target branch and open feature head. Keep maintenance separate from feature implementation. Parallel type and dependency work may use independent worktrees, but reconcile manifests and lockfiles in one integration tree; never let one worker's install mutate another worker's shared `node_modules`.
2. Reproduce the baseline with frozen installs and group TypeScript diagnostics by root cause. A faulty global index signature can generate many downstream errors; repair the declaration rather than patching each affected API. Inspect actual CI commands: transpilation/build success is not type safety.
3. Update compatible framework package groups together. For lockstep runtime/dev/serve packages, a staged install can produce ERESOLVE against old locked peers; try a normal compatible `npm update --package-lock-only --ignore-scripts --include=dev` before considering an override. Do not bypass peers with force/legacy flags. Verify registry latest for each manifest dependency independently because inherited npm configuration can omit development entries from `npm outdated`.
4. Make maintenance independently green: frozen installs, explicit typecheck, regression tests and affected package builds. Add regression tests to CI alongside typecheck so future dependency changes cannot bypass upgrade-sensitive import coverage. For Node VM-module tests, preserve the required invocation, such as `node --experimental-vm-modules --test tests/*.test.mjs`, and run the exact CI command locally.
5. Apply the pinned feature diff to a second isolated tree containing maintenance. Rebuild fixtures and rerun installed-SDK/browser acceptance, typecheck, unit tests and build. A conflict-free merge proves only text compatibility, not SDK/framework behavior. Retain exact approved exception assertions in raw results; fail on additional defects instead of widening the exception list.
6. Recommend merge order from the combined evidence. If maintenance merges first, refresh the feature against the resulting target and rerun CI. Its old green status does not prove compatibility with the new base. Distinguish local integration, hosted CI, Preview and live QA.
7. Report audit changes, unresolved findings, deferred majors and remaining UI checks. Qualify registry-latest counts with the checked dependency set and intentional runtime-major constraints; runtime-matched type packages need not track the newest registry major. Save raw before/after audits and distinguish inherited vulnerable package nodes from unique advisory roots. Fewer findings are not production security clearance.

## Baseline failure attribution

Reproduce failing browser fixtures and full normalized typecheck diagnostics on an independent pinned-base frozen install before declaring them pre-existing. Compare messages, not merely counts; retain raw output alongside normalized comparisons.

## Sanity media-plugin browser smoke

Use the real browser-bundled package, not a direct Node ESM import: CommonJS interop in browser dependencies can reject a Node import even when the Studio bundler works. Serve intercepted JavaScript explicitly as UTF-8 to avoid false syntax errors in Unicode-heavy bundles. Resolve plugin `tools` as either an array or a composable function before inspecting registration.

Successful import/tool registration is not authenticated asset-library browsing. Report that UI gap separately and never mutate CMS content just to validate an upgrade.
