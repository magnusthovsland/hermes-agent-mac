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

## Checkout-summary payment indicators

1. Separate login methods from payment methods before designing. Preserve the existing continuation buttons and handlers; a payment badge is informational, not a new login/payment action.
2. Use only confirmed available methods. A future-provider Figma example supplies visual inspiration, not proof that every depicted method is currently offered. Prefer a generic card symbol until supported card-network brands are confirmed.
3. For Wright's approved visual direction, retain the green summary panel and place a restrained, non-clickable payment row below the two login buttons, separated by a subtle rule. Use the heading `Betalingsmåter` and clarification `Du velger betalingsmåte senere.` Apply the same shared component to package and standalone-course summaries. The approved mockup uses Kort and Vipps; recheck availability before changing this set.
4. Produce actual desktop and narrow-mobile mockups, render and inspect both, and verify text/logo legibility and horizontal overflow. Use real approved brand assets for implementation, not a screenshot crop as a production logo.
5. For QA-only React Router presentation changes, read the existing root loader environment with `useRouteLoaderData` (type-only loader import) instead of reading `window` during render. This keeps SSR and hydration consistent without duplicating flags in sibling routes. Exercise QA, PROD, LOCAL and missing environment in SSR tests; use synthetic route data with the real shared component/CSS and real vendor web component for browser layout and captured-submit checks.
6. Report mockup approval, component implementation, PR creation and deployment separately. Keep this presentation-only change independent of event or attribution refactors when the user requests small PRs; protect existing callbacks and login-submit parameters in regression tests.

## Common pitfalls

- For Wright local read-only runtime comparisons, derive public Sanity settings from baseline `window.env` and inspect `wrightApi.ts` before setting `WRIGHT_API_URL`: it concatenates `/public/...`, so the base must have no trailing slash. A double-slash API URL can cause a false 500/empty office picker. Correct the harness and rerun before attributing failure to the PR. Use actual rendered office/course/package links, not filesystem route filenames (`/trafikkskole`, not `/avdeling`). Block browser non-GET requests, login handoffs and analytics; stop before summary continuation submissions. If the preview is protected, clearly label local built-app comparison as different evidence and record build provenance limitations.

- A route that works in Vite preview can still 404 on the hosting platform.
- Runtime `document.title` updates do not help non-JavaScript social crawlers.
- A sticky navigation card may look complete in a full-page capture while its final links are outside the actual viewport.
- Content-length conditionals in a shared legal-page component can make sibling pages visibly inconsistent. If the user asks for matching pages, use the same typography and spacing and let only the natural card height differ.
- A requested global contact value can remain stale if only one route is checked; search shared components and translation sources, then verify the exact formatted value in the live DOM.
- Reusing a visual title without parameterizing its heading level can break the semantic outline.
- A deployment hook may reject a valid branch when commit authorship is not recognized by the connected Git integration.
- Never select credentials or deploy hooks by frequency or nearby session text; use an explicitly named project mapping and verify the project before triggering it.

## Detailed references

- `references/vite-static-subroutes-vercel-and-long-toc.md` — finite Vite subroutes with raw-HTML metadata, real 404s, reusable product modules, long sticky legal-document TOCs, and deploy-hook verification.
- `references/intermittent-mobile-performance.md` — diagnose real-device slowness that synthetic tests miss: client-only blank shells, hashed-asset cache headers, redirects, carrier paths, rendering cost, and RUM evidence.
