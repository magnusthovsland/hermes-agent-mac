---
name: software-development-workflows
description: "Umbrella workflow for software engineering tasks: spikes, TDD, debugging, code review, simplification, codebase inspection, and language/runtime debuggers."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [software-development, debugging, testing, tdd, code-review, refactoring, codebase-inspection]
---

# Software Development Workflows

Use this skill for class-level software work: understanding a codebase, proving feasibility with a spike, writing tests first, debugging root causes, reviewing quality/security, simplifying recent changes, or attaching language/runtime debuggers.

## Triage: choose the mode

- **Unknown codebase or scope** → inspect structure, languages, LOC, dependencies, and test commands first.
- **Unproven approach** → run a time-boxed spike in an isolated scratch path; preserve only the learning.
- **Feature or bug with clear behavior** → use TDD: write/observe failing test, implement the smallest fix, refactor after green.
- **Bug without root cause** → use systematic debugging; reproduce, localize, explain mechanism, then fix.
- **Pre-commit or PR readiness** → request/recreate a code review: diff, tests, lint, security scan, docs impact, and auto-fix safe issues.
- **Messy recent changes** → simplify code with parallel review perspectives if appropriate.
- **Runtime-specific failure** → use Python `pdb`/`debugpy` or Node `--inspect`/Chrome DevTools Protocol.
- **Web app quality assessment** → run a dogfood/exploratory QA pass: plan scope, interact through the browser, collect screenshots/console evidence, categorize issues, and produce a structured bug report.
- **CMS/API access or content attribution question** → perform a read-only access/audit workflow: identify credential purpose without exposing secrets, query recent content, inspect transaction history, and map author IDs to users/robots.

## Operating rules

1. **Ground every claim in tools.** Read files, inspect git state, run tests, and capture exact failures.
2. **Do not skip reproduction.** A fix without a reproduced failure is a guess unless the user explicitly asks for a static patch only.
3. **Use the narrowest safe edit.** Avoid broad rewrites during debugging; refactor only after tests pass. Inspect and reuse existing functions/components before introducing parallel implementations. For this user's frontend work, favour changes to the existing startup and consent paths over new runtime components or dependencies when they already cover the responsibility.
4. **Verify at the right level.** Run the smallest relevant test first, then expand to the package/project checks. If typecheck or lint fails, rerun the identical command against the pinned unmodified base with the same runtime and lockfile before labeling errors pre-existing. Compare diagnostic messages, not just totals; normalize shifted line locations only for that comparison and retain the raw logs. A passing build does not imply a passing typecheck.
5. **Report durable evidence.** Include commands run and real pass/fail output in the final answer.
6. **Respect staged approval.** When this user requests a plan before the next job, present the scoped changes, reuse strategy, QA/PROD boundaries, tests and review/deployment gates in Norwegian, then wait for GO. Completing the preceding stage is not authorization for the next implementation; save only the plan while awaiting approval.
7. **Test the shipped SDK, not a mock of its internals.** Resolve the package entry point actually bundled by the app and exercise that installed artifact in a browser before accepting an integration. Source-tree private names may be renamed or absent in the distribution; do not build production adapters around them or hardcode minified replacements. Treat passing mocks as unit evidence only. For consent-sensitive SDKs, include failed/in-flight requests, delayed retries, withdrawal and re-consent in transport-intercepted tests, because stopping new capture does not establish that old queues stop sending. Do not label reset/shutdown as queue disposal without verifying its actual behavior; teardown can flush data. Keep any proposed replacement mechanism provisional until the real-artifact tests pass.
8. **Bound isolated payment acceptance tests.** Preserve prior JUnit XML before rerunning; exercise real orchestration and persistence with external payment/analytics boundaries mocked under network denial. Assert capture rejection/timeout suppresses purchases, replay recovers rather than trusting a prematurely terminal row, overlapping callback/polling deduplicates, and stale callbacks cannot regress state. Label mock capture markers as non-bank evidence; record the tested concurrency interleaving rather than claiming exhaustive race coverage. Compare original and copied production-source hashes to the manifest and stop only the isolated database, verifying both its scoped process status and closed loopback port.

## Multi-deliverable ownership

1. Before starting an approved batch, maintain one visible delivery ledger with a row per independently agreed deliverable: exact acceptance scope, local path/branch, dependency or open decision, verification evidence, and actual PR handle when created. Magnus expects ownership through PR creation, not repeated planning reports.
2. Reconcile the ledger when the user expands or splits scope. Move the relevant acceptance criteria explicitly; expanding an event specification must not make unfinished attribution disappear. Preserve parked work without treating it as authorized active work.
3. When asked to complete only a numbered stage and defer the rest, execute only that stage through its stated deliverable. Do not continue research or refactors for deferred tasks merely because they were previously authorized.
4. Deliver independent approved changes while a different workstream awaits a decision. Do not let an experimental integration hold a small UI or instrumentation PR hostage; identify genuine shared-file conflicts and test the combined changes locally before recommending merge order.
5. Use explicit states: specification agreed, implemented locally, tests verified, PR created, merged, deployed and live-verified. Check the actual artifact or remote target before reporting each transition; approval of a design is not implementation, and a tested partial implementation is not the full agreed feature.
6. End a delivery report with verified PR identifiers and remaining blockers. If no PR exists, say so plainly. Do not call work ready while named acceptance criteria remain unimplemented, or ask the user to re-authorize unchanged work already explicitly approved.

## Labeled playbooks

### React Router/Sanity SEO metadata regression audits

For Sanity-backed React Router sites where `<title>`, meta description, or Open Graph tags disappear from live source, do a layered read-only audit before blaming CMS content: verify schema fields, verify production documents still contain `seoTitle`/`seoDescription`, verify frontend GROQ queries fetch those fields, then inspect route `meta()` rendering. React Router v8 removed the old `data` argument for `meta()`; migrate `({ data })` to `({ loaderData })` and `matches[*].data` to `matches[*].loaderData`. Full checklist: `references/react-router-sanity-seo-metadata-audit.md`.

### Codebase inspection

Use `pygount` or language-native tools to quantify files, languages, generated/vendor ratios, and hotspots. The preserved package `references/packages/codebase-inspection/` contains the original commands and report style.

### Spike

A spike is disposable experimentation to reduce uncertainty, not an architectural commitment.

1. Write the concrete observed gap and compare the existing implementation, documented vendor mechanisms and the smallest adapter before proposing new shared infrastructure.
2. Separate user-approved requirements, platform contracts and engineering assumptions. Obtain a decision before turning an assumed retention, source-priority or consent rule into a release-blocking requirement.
3. Keep the experiment isolated and time-boxed. Test the intended behavior, but also challenge whether that behavior actually serves the user's requirement; passing tests can faithfully validate the wrong source-priority rule.
4. Label the result as a documented standard, a proven local technique or a custom hypothesis. Never call a home-grown prototype best practice solely because its tests pass. Report expected failures as unresolved acceptance, not a green delivery.
5. Conclude with the smallest justified production change and its remaining decisions. Do not promote custom persistence, identity, cross-tab coordination or restart behavior merely because it has already been written. Preserve stable behavior and independent deliverables while evaluating the tradeoff.

### Test-driven development

Follow RED → GREEN → REFACTOR. Create the failing test before implementation when behavior is knowable. Do not broaden refactors until the targeted test is green.

### Systematic debugging

Use a four-phase loop: reproduce, localize, explain, fix/verify. Record hypotheses and falsification evidence; avoid patching symptoms.

### Code review and simplification

For pre-commit review, inspect diff and run quality gates. Magnus wants critical comparison with existing implementations (especially Teoria for Wright analytics), but not deletion of necessary safeguards merely to reduce line count.

1. Pin both implementations to remote commit SHAs and establish their actual initialization paths, configuration and installed dependency versions. Keep comparison snapshots separate from a worktree being edited in parallel, so the evidence cannot silently change underneath the review.
2. Compare requirements before comparing code size: authenticated versus anonymous identity, collection on rejection versus no collection, reload behavior, persistence and cancellation guarantees. Treat static absence of a safeguard as a test question, not a reproduced defect in the reference product.
3. Identify removable branches, duplicated responsibilities and unnecessary abstractions; preserve regression tests for demonstrated failures. Do not import another product's auth, cookieless policy or extra configuration layers merely for consistency.
4. Run a bounded simplification spike against the same acceptance suite. If only a small control-flow cleanup is safe, say so; do not describe a two-line reduction as architectural simplification or justify current complexity with unspecified future features.
5. Rebuild browser fixtures from the changed source into a fresh output directory before testing, because a stale bundle can falsely validate a refactor. Run existing tests as well as new tests and compare full normalized typecheck diagnostics with the pinned baseline.
6. Report the necessary behavior, actual simplification, test evidence and delivery state separately. Distinguish application-code volume from tests/documentation. If satisfying the agreed requirements demands substantially more complexity or a new user-visible tradeoff, pause and explain it before expanding the implementation.

### Web application dogfooding / exploratory QA

For browser-facing apps, use a structured dogfood pass: define scope, explore primary flows, capture screenshots/console/network evidence, categorize defects by user impact, and produce an actionable report. The preserved `dogfood` package includes an issue taxonomy and report template.

### React Router landing-page Lighthouse audits

For read-only Lighthouse investigations on React Router/Vite/Vercel marketing pages, measure production at least twice, identify the actual LCP node before accepting the hypothesis, inspect emitted head order and headers, separate app code from proxied analytics, and compare production against source without assuming they match. **Never label a local Lighthouse CLI run as PageSpeed Insights:** report hosted PSI, local CLI, and field/RUM results separately, capture the CLI environment, and use hosted PSI as the governing lab baseline when the business question concerns Google's hosted result. Lead the report with a compact in-chat conclusion before the detailed evidence. See `references/react-router-lighthouse-landing-page-audit.md` for the React Router audit workflow and `references/web-performance-lab-audits.md` for PSI-vs-CLI measurement discipline, production/source discrepancies, Astro/islands migration evaluation, and score-prediction rules.

### Frontend framework upgrade regressions

When a frontend regression appears after a framework/dependency upgrade, verify both the live output and the underlying data before blaming the CMS. For React Router v8 specifically, route `meta()` functions must use `loaderData` instead of the deprecated v7 `data` argument; stale `({ data })` usage can silently remove server-rendered `<title>`, meta description, Open Graph, and Twitter tags even when CMS SEO fields still exist. See `references/react-router-v8-route-meta.md` for the reproduction and fix pattern.

### White-label CSS theme regressions

When several tenant sites lose brand colors but layout/content still render, trace the tenant theme class, scoped CSS custom properties, and server-rendered theme data before blaming the stylesheet. Blank strings bypass nullish fallbacks and can silently invalidate every token-backed color declaration. See `references/white-label-css-theme-regression.md` for the live-DOM, streamed-loader-data, deployment-correlation, reversible proof, and hardening workflow.

### CMS/API content access audits

When asked whether an agent has CMS access or who made recent content changes, keep the workflow read-only by default: find credential metadata, query current content, inspect transaction history for attribution, and map author IDs to humans/robots. See `references/sanity-content-lake-audit.md` for the Sanity Content Lake/History API pattern and reporting pitfalls. For Teoria-specific Sanity fact-box/published-vs-draft audits, use `references/teoria-sanity-factbox-audit.md`; it captures the Infinity Drift project ID, frontend data flow, localization-validation pitfall, and “40 % stryker” correction.

### Sanity navigation/footer link modeling

When a Sanity-managed footer/header link needs to point at a frontend route that is not a Sanity document, inspect the schema and frontend renderer before recommending workarounds. Do not create fake Sanity pages solely to satisfy an internal reference. Prefer existing `linkExternal` as an immediate workaround for same-domain non-Sanity routes, and recommend/implement a `linkRelative` type for the long-term model when appropriate. See `references/sanity-footer-link-modeling.md` for the decision order and editor guidance.

### Multi-tenant analytics and purchase attribution

When a tenant-owned marketing site hands a user into a separate SaaS app for registration/payment, first distinguish GA4 reporting, GA4 source attribution, direct Google Ads conversion upload, and generic data-layer delivery. Do not automatically prescribe cross-domain GA4 for every tenant. A purchase event alone does not preserve organic/paid source; carry tenant-scoped GA4 identity/session context through a signed attribution handoff and emit an idempotent backend-confirmed purchase through a browser or Measurement Protocol adapter. Browser `gtag` requires a tenant Measurement ID but no API secret; server-side Measurement Protocol requires both. See `references/multi-tenant-purchase-attribution.md`.

### Server-side conversion tracking audits

For read-only audits spanning payment callbacks, PostHog identity, CMP consent, attribution, proxy IP and advertising delivery, use `references/server-conversion-tracking-audit.md`. Trace actual capture ordering and existing idempotency before making claims; distinguish repository evidence from deployed headers, SDK behavior and externally configured destinations.

### Isolated attribution acceptance tests

For dependency-free semantic UI checks in a Vite/Oxc repository, transpile the actual TSX with the installed Vite transform, evaluate it with Node VM modules, and invoke handlers from returned React elements. Mock router/hooks/SDK boundaries explicitly, execute mount effects to catch accidental mount captures, and simulate nested bubbling with the same `defaultPrevented` event object. Test both responsive branches and unchanged submit payloads. Label this handler-level evidence, not browser rendering or native keyboard/E2E proof; document the required `--experimental-vm-modules` flag.

For tracking audits without safe staging, execute actual source functions in a network-denied local harness and assert desired acceptance behavior; never count tests that merely affirm a known defect as acceptance passes. Match the repository Node engine and direct lockfile dependency versions, verify official runtime checksums, record source hashes/commands/raw outcomes, and distinguish minimal dependency subsets from full frozen installs. On macOS, `sandbox-exec -p '(version 1)(allow default)(deny network*)'` can enforce denial beyond mocked fetch; verify it with a loopback-only socket probe expecting EPERM. Exercise the purchase caller as well as its tracking helper: a helper accepting optional event parameters does not prove that the real caller supplies stable transaction ID, value and currency. Label mocked auth/session/UI/effects explicitly and never equate function-level results with browser/CMP/payment/Ads E2E or deployed behavior.

### External ERP/API integration research

When researching a business-system API integration, distinguish **the business action** from **the bookkeeping mirror** before recommending mutations. For financial flows, explicitly separate “system should initiate money movement” from “money already moved elsewhere; system should only record it,” because using a payout API after an external PSP refund can create duplicate payments. For Visma Business NXT customer refunds, see `references/visma-business-nxt-customer-refunds.md` for the durable GraphQL patterns and pitfalls.

### OpenClaw / agent gateway runtime debugging

When an agent gateway is reachable but chat replies fail, debug in layers: process/HTTP health, channel transport, then model runtime/auth with a tiny `openclaw agent` healthcheck. Do not declare success from `channels status` alone. After `doctor --fix` or updates, watch for legacy `openai-codex/*` → `openai/*` migration problems where routes are repaired but per-agent OpenAI OAuth profiles are missing from `openclaw-agent.sqlite`. Use `references/openclaw-gateway-runtime-auth-troubleshooting.md` for the exact triage, repair, backup, and verification sequence.

### SMS provider migration audits

For read-only SMS integration inventories and provider replacement assessments, use `references/sms-provider-migration-audit.md`. It covers call-site counting, effective webhook paths, tenant/office number routing, DLR/idempotency, QA safety, redaction and line-reference verification.

### Python debugging

Use `pdb` for local interactive stepping and `debugpy` when a long-running process or test needs DAP-style attach. See `references/packages/python-debugpy/` for command recipes.

### Node debugging

Use `node --inspect` or `--inspect-brk`, connect to the Chrome DevTools Protocol, set breakpoints, evaluate expressions, and continue. See `references/packages/node-inspect-debugger/`.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `codebase-inspection`, `dogfood`, `node-inspect-debugger`, `python-debugpy`, `requesting-code-review`, `simplify-code`, `spike`, `systematic-debugging`, and `test-driven-development`.
