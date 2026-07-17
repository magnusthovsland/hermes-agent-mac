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
3. **Use the narrowest safe edit.** Avoid broad rewrites during debugging; refactor only after tests pass.
4. **Verify at the right level.** Run the smallest relevant test first, then expand to the package/project checks.
5. **Report durable evidence.** Include commands run and real pass/fail output in the final answer.

## Labeled playbooks

### React Router/Sanity SEO metadata regression audits

For Sanity-backed React Router sites where `<title>`, meta description, or Open Graph tags disappear from live source, do a layered read-only audit before blaming CMS content: verify schema fields, verify production documents still contain `seoTitle`/`seoDescription`, verify frontend GROQ queries fetch those fields, then inspect route `meta()` rendering. React Router v8 removed the old `data` argument for `meta()`; migrate `({ data })` to `({ loaderData })` and `matches[*].data` to `matches[*].loaderData`. Full checklist: `references/react-router-sanity-seo-metadata-audit.md`.

### Codebase inspection

Use `pygount` or language-native tools to quantify files, languages, generated/vendor ratios, and hotspots. The preserved package `references/packages/codebase-inspection/` contains the original commands and report style.

### Spike

A spike is disposable experimentation to reduce uncertainty. Keep it isolated, time-boxed, and conclude with what was learned, what failed, and whether to proceed.

### Test-driven development

Follow RED → GREEN → REFACTOR. Create the failing test before implementation when behavior is knowable. Do not broaden refactors until the targeted test is green.

### Systematic debugging

Use a four-phase loop: reproduce, localize, explain, fix/verify. Record hypotheses and falsification evidence; avoid patching symptoms.

### Code review and simplification

For pre-commit review, inspect diff and run quality gates. For simplification, ask what code can be deleted, what abstractions can collapse, and whether behavior remains covered by tests.

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

### External ERP/API integration research

When researching a business-system API integration, distinguish **the business action** from **the bookkeeping mirror** before recommending mutations. For financial flows, explicitly separate “system should initiate money movement” from “money already moved elsewhere; system should only record it,” because using a payout API after an external PSP refund can create duplicate payments. For Visma Business NXT customer refunds, see `references/visma-business-nxt-customer-refunds.md` for the durable GraphQL patterns and pitfalls.

### OpenClaw / agent gateway runtime debugging

When an agent gateway is reachable but chat replies fail, debug in layers: process/HTTP health, channel transport, then model runtime/auth with a tiny `openclaw agent` healthcheck. Do not declare success from `channels status` alone. After `doctor --fix` or updates, watch for legacy `openai-codex/*` → `openai/*` migration problems where routes are repaired but per-agent OpenAI OAuth profiles are missing from `openclaw-agent.sqlite`. Use `references/openclaw-gateway-runtime-auth-troubleshooting.md` for the exact triage, repair, backup, and verification sequence.

### Python debugging

Use `pdb` for local interactive stepping and `debugpy` when a long-running process or test needs DAP-style attach. See `references/packages/python-debugpy/` for command recipes.

### Node debugging

Use `node --inspect` or `--inspect-brk`, connect to the Chrome DevTools Protocol, set breakpoints, evaluate expressions, and continue. See `references/packages/node-inspect-debugger/`.

## Preserved source packages

Full prior skill packages are preserved under `references/packages/`: `codebase-inspection`, `dogfood`, `node-inspect-debugger`, `python-debugpy`, `requesting-code-review`, `simplify-code`, `spike`, `systematic-debugging`, and `test-driven-development`.
