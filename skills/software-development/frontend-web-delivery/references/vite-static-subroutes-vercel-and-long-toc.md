# Vite static subroutes on Vercel and long legal-document navigation

Use this recipe when adding a finite set of public policy, terms, product, or documentation subroutes to a small Vite/React site that does not otherwise need a router or SSR.

## Reuse existing page modules without copying

- Extract a homepage product block into a shared component before reusing it on a product subroute.
- Parameterize only real differences, such as CTA destination and heading level.
- Preserve the semantic outline: the homepage instance may need an `h3`, while the standalone product page needs the same visual title rendered as its single `h1`.
- On subroutes, make one-page navigation links point to `/#section`, not a local `#section`.
- If legal content is Norwegian-only, hide the language toggle on that route rather than producing a mixed-language document whose `<html lang>` claims otherwise.

## Give direct routes real HTML metadata

A wildcard rewrite from every subroute to one `index.html` is convenient but creates two durable problems:

1. non-JavaScript crawlers and social previews receive homepage title/Open Graph data;
2. unknown descendants become soft 404 responses with HTTP 200.

For a small finite route set, prefer build-time route HTML generation:

1. Run the normal Vite build.
2. Read `dist/index.html` in a post-build Node script.
3. For each route, replace title, description, Open Graph, Twitter, and canonical values.
4. Write clean-URL-compatible files such as `dist/product.html` and `dist/product/privacy.html`.
5. Set Vercel `cleanUrls: true` and avoid a broad wildcard rewrite.

The React app can still select content from `window.location.pathname`; route-specific HTML exists for correct initial metadata and HTTP routing.

Verify deployed **raw HTML**, not only the hydrated DOM:

- every known route returns HTTP 200;
- raw source contains the correct title, description, canonical, and `og:url`;
- an unknown descendant returns a real HTTP 404;
- the homepage remains HTTP 200 with its own metadata.

## Keep a long sticky TOC fully visible

A sticky table of contents can be taller than the viewport even when all items appear in a full-page screenshot. Reproduce at a realistic short desktop viewport and measure after sticky positioning engages:

```js
document.documentElement.style.scrollBehavior = 'auto';
window.scrollTo(0, 900);
const nav = document.querySelector('nav[aria-label="Innhold"]');
const rect = nav.getBoundingClientRect();
const last = nav.querySelector('li:last-child').getBoundingClientRect();
({
  viewport: innerHeight,
  navHeight: rect.height,
  available: innerHeight - rect.top,
  allVisible: rect.bottom <= innerHeight,
  lastVisible: last.bottom <= innerHeight,
});
```

For longer documents, compact only the desktop TOC:

- reduce card padding;
- reduce row spacing;
- use a smaller font and line height;
- leave article typography unchanged.

Prefer compaction over an inner scrollbar when the list can reasonably fit. The final section link must remain visible while the card is sticky. **When sibling legal pages are meant to share one visual design, apply the same TOC typography, padding, line height, and row spacing to all of them.** The shorter document will naturally produce a shorter card; do not make it visually roomier through a content-length conditional unless the user explicitly requests that difference. After changing the shared component, compare computed styles on every sibling route. On mobile, use a collapsed native `details` element instead of forcing users to scroll through the entire list before reaching the article.

## Shared legal-page consistency

- Treat sibling privacy and terms pages as a single visual system. A size or spacing change requested for one should be checked and normally applied through their shared component so the pages do not diverge.
- Compare actual computed values—font size, line height, padding, and row spacing—rather than assuming reuse of the same component guarantees identical rendering.
- Search global values such as phone numbers across shared components and translation sources. Apply the exact requested formatting and verify it in the production DOM after deployment, not merely in the source diff.

## Deploy-hook workflow

When Vercel auto-deploy is disabled:

1. Configure repository-local Git author name/email to an identity authorized for the connected Git repository **before** committing.
2. Build, commit, and push the production branch.
3. Trigger the specifically named deploy-hook credential for that project. Never infer a hook from nearby transcript context or occurrence frequency; several projects may coexist in one session archive.
4. Wait for CI and deployment completion.
5. Detect the new live build by emitted asset hash or route-specific HTML.
6. Run live HTTP, raw-metadata, and browser QA.

If a deploy hook responds with `incorrect_git_source_info`, check both hook-to-project/branch mapping and commit author identity. Do not expose hook URLs or tokens in logs, commits, or responses.
