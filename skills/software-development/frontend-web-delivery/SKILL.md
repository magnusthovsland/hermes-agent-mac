---
name: frontend-web-delivery
description: "Use when extending or shipping public frontend sites."
version: 1.0.0
metadata:
  hermes:
    tags: [frontend, vite, react, vercel, accessibility, deployment]
---

# Frontend Web Delivery

Use this class-level workflow for extending and publishing public-facing frontend sites: landing pages, product subroutes, legal/documentation pages, static metadata, responsive navigation, CI, and production deployment.

## Core workflow

1. **Inspect before editing**
   - Confirm framework, build scripts, routing model, deployment config, current branch, remotes, and clean working tree.
   - Identify reusable components and existing visual/typographic conventions.
2. **Choose the smallest architecture change**
   - Preserve a lightweight one-page site when only a few subroutes are needed.
   - Extract shared UI instead of duplicating homepage markup.
   - Before reusing a leaf component outside its original parent, inspect CSS ancestor selectors. If styles depend on a collection wrapper, move the component's own styles to its local class without changing design values; keep collection layout separate. Verify primary and secondary variants plus the new use with computed styles at desktop/mobile widths. For isolated checks, render actual components with clearly labeled synthetic fixtures, remove temporary harness files before committing, and do not present fixture verification as deployed-site acceptance.
   - Add a full router or SSR only when route count, navigation behavior, or server rendering actually requires it.
3. **Design for semantics, consistency, and responsive use**
   - Maintain one `h1` per page and correct nested heading levels.
   - Treat sibling legal/document pages as one visual system. Apply shared typography and spacing in their shared component, then compare computed styles on every sibling route.
   - Make subpage navigation point back to homepage anchors correctly.
   - Verify both desktop and a real narrow mobile viewport; full-page screenshots alone do not prove sticky or viewport-constrained behavior.
4. **Implement correct direct-route delivery**
   - Distinguish hydrated DOM metadata from metadata present in raw HTML.
   - Avoid wildcard rewrites that turn unknown paths into soft 404s unless that behavior is intentional.
5. **Verify locally and diagnose real mobile delivery**
   - Run type checking, the production build, diff checks, and direct-route HTTP checks.
   - Inspect desktop and mobile rendering, heading hierarchy, overflow, internal links, title, description, canonical, and Open Graph values.
   - For intermittent mobile slowness, separate origin latency, redirect/TLS cost, asset delivery, JavaScript startup, and post-load rendering cost.
   - Inspect raw HTML for an empty client-only application shell, and verify that content-hashed JS/CSS use long-lived immutable caching rather than forced revalidation.
   - Treat Lighthouse as synthetic evidence, not as a refutation of real-device reports; correlate exact timestamps, carrier/Wi-Fi, device, and browser through real-user monitoring.
6. **Publish safely**
   - Use the repository identity expected by the Git/deployment integration before creating commits.
   - Push only after required approval, monitor CI, trigger the correct project-specific deployment mechanism, and verify the actual live build.
7. **Verify production, not intent**
   - Confirm known routes return 200, unknown routes return the expected status, raw HTML metadata is route-correct, and the homepage has not regressed.
   - Detect deployment completion using the emitted asset hash or route HTML rather than waiting blindly.
   - When a QA domain is reassigned to a feature Preview, verify the exact hostname, selected Git branch and the deployment's Domains list before diagnosing stale content as DNS propagation. Distinguish a provider-side branch/domain assignment from an actual DNS-record change. After the assignment is saved, an authorized redeploy of the intended branch can refresh the association; require Ready and then verify the custom hostname serves that build. Do not infer the code or API environment from a hostname containing `qa`, because it may now point to a feature deployment.

## Technical metadata and summary-heading fixes

1. Separate HTML defects from intentional SEO policy before proposing a patch. Inspect the shared metadata helper, route-level canonical construction and robots directives; a course or package URL deliberately canonicalized to its office page is not automatically an error. Magnus requires these deliberate targets to remain unchanged unless he explicitly approves a policy change.
2. For a technical canonical repair, preserve the destination pathname and meaningful query parameters. Correct the rendered element and, where confirmed wrong, the public origin without replacing every route with a self-canonical or stripping queries globally. Reuse the existing public-origin configuration; do not infer the intended public domain from an internal deployment hostname or an untrusted forwarded header.
3. Test the actual installed router's metadata rendering in raw SSR HTML as well as the hydrated DOM. Assert a single `link[rel="canonical"]`, no `meta[rel="canonical"]`, the intended destination and unchanged robots directives; descriptor-shaped unit assertions alone can miss the wrong emitted tag.
4. Treat Wright's specific-course/package summary as a transaction page, not an SEO landing page. Keep `noindex, nofollow` and its intentional canonical target. Use `Oppsummering` as its only H1 and the course/package name as a subordinate heading; inspect existing markup first and do not alter a sibling summary that already has the correct hierarchy.
5. Preserve visible wording, classes, layout, continuation handlers and payment information when changing heading tags. Check tag-dependent CSS and mobile/desktop appearance; describe this as semantic HTML cleanup, not a critical SEO or checkout failure.
6. Keep the resulting PR small and separate from CMP reload behavior, tracking, dependency upgrades and indexing-policy changes. When Magnus requests a proposal first, enumerate the technical changes and preserved behavior, then wait for approval before editing code.

## Checkout-summary payment indicators

1. Separate login methods from payment methods before designing. Preserve the existing continuation buttons and handlers; a payment badge is informational, not a new login/payment action.
2. Use only confirmed available methods. A future-provider Figma example supplies visual inspiration, not proof that every depicted method is currently offered. Prefer a generic card symbol until supported card-network brands are confirmed.
3. For Wright's approved visual direction, retain the green summary panel and place a restrained, non-clickable payment row below the two login buttons, separated by a subtle rule. Use the heading `Betalingsmåter` and clarification `Du velger betalingsmåte senere.` Apply the same shared component to package and standalone-course summaries. The approved mockup uses Kort and Vipps; recheck availability before changing this set.
4. Produce actual desktop and narrow-mobile mockups, render and inspect both, and verify text/logo legibility and horizontal overflow. Use real approved brand assets for implementation, not a screenshot crop as a production logo.
5. For QA-only React Router presentation changes, read the existing root loader environment with `useRouteLoaderData` (type-only loader import) instead of reading `window` during render. This keeps SSR and hydration consistent without duplicating flags in sibling routes. Exercise QA, PROD, LOCAL and missing environment in SSR tests; use synthetic route data with the real shared component/CSS and real vendor web component for browser layout and captured-submit checks.
6. Report mockup approval, component implementation, PR creation and deployment separately. Keep this presentation-only change independent of event or attribution refactors when the user requests small PRs; protect existing callbacks and login-submit parameters in regression tests.

## Frontend-only package campaign pricing

1. Establish the repository boundary before editing. For Magnus' White Label price work, change only `infinity-wl-web`; use Wright as a read-only reference, not a second implementation target. Do not change Ovio APIs, discount rules, checkout, course-date cards or direct course sales. A missing field in one response is not permission to widen scope.
2. Trace the working reference from UI through loader to the exact endpoint, then compare live list and detail response shapes. Reuse an existing endpoint and fetch helper before proposing a backend addition; shared APIs can expose different fields on list and detail resources. Magnus prefers exhausting existing API contracts before considering an API change; read backend DTO/mapping/controller code when needed, but treat that as investigation, not permission to modify the backend.
   - Enumerate entry paths as well as components: a package selected first and a course that is required to purchase a package can reach different loaders and price mappings. Check both paths with the same course/date/package before claiming package-price coverage; keep genuine standalone-course sales separate.
   - For a course-to-required-package path, establish an explicit authoritative package ID and requirement flag from existing contracts before proposing reuse of package pricing or navigation. Compare raw course detail, office-product detail/list and package detail/list under the same tenant and office, then trace how the backend selects the displayed name and price. A package-like course name, matching amount, matching contents or `shouldSuggestCourse` is not proof that the package is mandatory. Do not hardcode the observed package ID or select the first containing package.
   - Report available campaign amounts separately from availability of the required-package relationship. If the relationship is not yet established, state that gap rather than promising a frontend-only implementation or declaring an API addition necessary. Once established and implementation is authorized, reuse the existing package price mapper, preserve the selected course/date and continuation identifiers, and leave ordinary course behavior unchanged.
3. Display a campaign only when `hasAutomaticDiscount` is true and both `packagePrice` and `discountedPrice` exist. Use the API's discounted amount, normal-package before-price and their difference; never derive a discount rate or substitute `originalPrice`, which represents package-content value. Treat zero as present, not missing. Do not impose a stricter discounted-less-than-base eligibility rule: equal API prices still honor the automatic flag, but display a savings label only for a positive saving. Preserve the legacy 100 kr savings-label threshold for noncampaign packages and existing fallback price behavior when the flag is false or fields are unavailable.
4. If detail needs list enrichment, keep the same tenant and use authoritative office/product identifiers from the detail response. Match the exact office-package and product IDs; do not take the first result, guess from the first certificate, or borrow another office's price. Merge pricing fields only so rich detail, selected course and handoff parameters remain intact. Make optional enrichment failure fall back safely.
5. Reuse existing price/tag and summary components, with optional campaign inputs that leave standalone-course defaults unchanged. For Magnus' White Label cards with angled badges, keep the visual order **Nå badge → Spar badge → smaller Førpris text with the amount struck through**. Place before-price below both badges, not between them; keep the purchase summary's existing order unchanged. Implement this as a markup reorder in the existing tag stack, not a new component or pricing change. Test card, detail and summary, both before and after course-date selection; a shared component alone does not prove each loader supplies the fields.
6. Verify absent fields, partial fields, zero discounted price, unused manual codes, false automatic flag, wrong package/office and failed enrichment. Render actual components/styles in labeled synthetic fixtures and intercept continuation submissions. For animated cards, assert opacity is 1 before screenshots; waiting alone can capture a partially transparent animation. Keep temporary fixtures outside production source.
7. Distinguish UI fixtures, public quotation parity and authenticated checkout verification. Inspect the endpoint's implementation before invoking a checkout-named POST; do not create orders, reservations or payment sessions to prove a display amount. Deliver the requested draft PR with screenshots and explicit remaining verification gates, without merging or rolling out.

See `references/package-campaign-pricing.md` for endpoint tracing, money-field mapping and the read-only quote verification boundary.

## Forced-package course purchase pricing

- Inspect both the public course mapper and checkout resolver before trying to map a course to a forced package in the frontend. Public course/office-product responses may omit the forced-package ID even though the backend substitutes package displayName and ordinary price. Package-content membership or equal names never proves the configured forced-package relationship.
- A frontend-only option is the existing anonymous quote `POST /api/public/checkout` with `item.courseId` and `item.officeProductId: null`, tenant header and valid certificate selection. Source-check that this is calculation-only before calling it; the backend then resolves the configured package. Preserve course-ID handoff rather than guessing a package ID. Never substitute authenticated checkout or order-completion endpoints.
- For automatic campaign display, inspect the course DTO's price provenance: in the verified contract it supplies ordinary package price when forced, while quote `toBePaidAmount` supplies the effective amount and `discount.type` identifies AUTOMATIC. These can provide before/now without an API change, provided both responses refer to the same course/tenant. Do not use quote `valueAmount` or `discount.amount` as campaign before-price/savings: they include content-value/package savings. Compute only the display difference of verified API amounts. Validate ordinary courses, no discount, zero values, missing quote and course-date preservation; distinguish public quote tests from authenticated checkout.

## Deployment installation and lockfile diagnosis

1. Locate the first fatal step in the deployment log before changing application code. Separate initial dependency installation, application compilation and any subsequent function-packaging installation. A successful local production build does not exercise every hosted packaging step; report each verified stage separately.
2. Compare `package.json`'s `packageManager`, `pnpm-workspace.yaml`, `.npmrc`, lockfile settings/overrides and `vercel.json` install/runtime settings. Read the package-manager version used at the failing step rather than assuming it matches the first install. Compare these files against the feature's base before attributing the failure to new application code.
3. Reproduce configuration mismatches in separate scratch directories containing copies of the manifest, lockfile and workspace configuration. Run `corepack pnpm@<version> install --frozen-lockfile --lockfile-only --ignore-scripts` with the failing version and the declared version. Preserve stdout, exit codes and inputs; never regenerate the repository lockfile merely to investigate. This isolates lockfile validation, not a complete deployment.
4. When overrides are present in the lockfile and workspace configuration but the older client rejects them, test a causal control: mirror the exact overrides into `package.json` under `pnpm.overrides` in a third disposable copy and repeat the older-client command. A failing older-client baseline, passing declared-client baseline and passing mirrored control distinguish configuration-location compatibility from a corrupt lockfile. Keep this diagnostic compatibility shim out of production unless separately justified.
5. Preserve security overrides and frozen-lockfile validation while proposing a correction. Verify current provider documentation and the actual project settings before recommending Corepack/version-selection changes; do not present a candidate environment setting as a proven deployment fix. Inspect explicit `functions.<path>.runtime` pins in `vercel.json`: an older pinned builder can perform its own dependency installation after the framework build and use a different package-manager version. Where the function supports provider-default detection, removing an obsolete explicit runtime pin is a narrow candidate fix, not a reason to delete the function or rewrite its handler. Obtain authorization, test the changed configuration in hosted Preview, and require Ready before declaring the packaging failure resolved; a local framework build alone is insufficient.
6. If the deployment fix already exists on another branch, use the branch-refresh and Preview-isolation procedure in `github-workflows` rather than making another speculative dependency change. Keep the fix's observed hosted outcome distinct from unverified runtime/UI acceptance.

## Common pitfalls

- For Wright local read-only runtime comparisons, derive public Sanity settings from baseline `window.env` and inspect `wrightApi.ts` before setting `WRIGHT_API_URL`: it concatenates `/public/...`, so the base must have no trailing slash. A double-slash API URL can cause a false 500/empty office picker. Correct the harness and rerun before attributing failure to the PR. Use actual rendered office/course/package links, not filesystem route filenames (`/trafikkskole`, not `/avdeling`). Block browser non-GET requests, login handoffs and analytics; stop before summary continuation submissions. If the preview is protected, clearly label local built-app comparison as different evidence and record build provenance limitations.

- A route that works in Vite preview can still 404 on the hosting platform.
- Runtime `document.title` updates do not help non-JavaScript social crawlers.
- A sticky navigation card may look complete in a full-page capture while its final links are outside the actual viewport.
- For rotated badges followed by padded text, measure the visible text with `document.createRange()`, `selectNodeContents(element)` and `getBoundingClientRect()`, then inspect the screenshot. A transformed element's axis-aligned box includes padding and empty corners, so overlapping wrapper bounds alone do not establish a visible text collision. Assert content order, readable text clearance and no horizontal overflow on desktop and mobile.
- Content-length conditionals in a shared legal-page component can make sibling pages visibly inconsistent. If the user asks for matching pages, use the same typography and spacing and let only the natural card height differ.
- A requested global contact value can remain stale if only one route is checked; search shared components and translation sources, then verify the exact formatted value in the live DOM.
- Reusing a visual title without parameterizing its heading level can break the semantic outline.
- A deployment hook may reject a valid branch when commit authorship is not recognized by the connected Git integration.
- Never select credentials or deploy hooks by frequency or nearby session text; use an explicitly named project mapping and verify the project before triggering it.

## Detailed references

- `references/vite-static-subroutes-vercel-and-long-toc.md` — finite Vite subroutes with raw-HTML metadata, real 404s, reusable product modules, long sticky legal-document TOCs, and deploy-hook verification.
- `references/intermittent-mobile-performance.md` — diagnose real-device slowness that synthetic tests miss: client-only blank shells, hashed-asset cache headers, redirects, carrier paths, rendering cost, and RUM evidence.
