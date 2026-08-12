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
   - Add a full router or SSR only when route count, navigation behavior, or server rendering actually requires it.
3. **Design for semantics, consistency, and responsive use**
   - Maintain one `h1` per page and correct nested heading levels.
   - Treat sibling legal/document pages as one visual system. Apply shared typography and spacing in their shared component, then compare computed styles on every sibling route.
   - Make subpage navigation point back to homepage anchors correctly.
   - Verify both desktop and a real narrow mobile viewport; full-page screenshots alone do not prove sticky or viewport-constrained behavior.
4. **Implement correct direct-route delivery**
   - Distinguish hydrated DOM metadata from metadata present in raw HTML.
   - Avoid wildcard rewrites that turn unknown paths into soft 404s unless that behavior is intentional.
5. **Verify locally**
   - Run type checking, the production build, diff checks, and direct-route HTTP checks.
   - Inspect desktop and mobile rendering, heading hierarchy, overflow, internal links, title, description, canonical, and Open Graph values.
6. **Publish safely**
   - Use the repository identity expected by the Git/deployment integration before creating commits.
   - Push only after required approval, monitor CI, trigger the correct project-specific deployment mechanism, and verify the actual live build.
7. **Verify production, not intent**
   - Confirm known routes return 200, unknown routes return the expected status, raw HTML metadata is route-correct, and the homepage has not regressed.
   - Detect deployment completion using the emitted asset hash or route HTML rather than waiting blindly.

## Common pitfalls

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
