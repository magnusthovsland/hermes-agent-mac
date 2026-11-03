# React Router + Sanity SEO metadata audit pattern

Use when a Sanity-backed React Router/Remix-style site appears to have lost `<title>`, meta description, or Open Graph tags after dependency/CMS work.

## Key diagnostic lesson

Do not assume SEO text was deleted from Sanity just because live HTML lacks meta tags. Separate three layers:

1. **CMS schema** — do fields such as `seoTitle`, `seoDescription`, `seoImage` still exist and are they attached to page document schemas?
2. **CMS content** — do production documents still contain values for those fields?
3. **Frontend rendering** — do route queries fetch those fields and do route `meta()` functions actually render them into source HTML?

## React Router v8 pitfall

React Router v8 removed the deprecated `data` field passed to route `meta()` functions. Code migrated from v7 may still compile or type loosely but return empty meta arrays at runtime if it does:

```ts
export const meta: MetaFunction<typeof loader> = ({ data }) => {
  if (!data) return [];
  return getSeoMetaData({ title: data.page.seoTitle });
};
```

Update to `loaderData`:

```ts
export const meta: MetaFunction<typeof loader> = ({ loaderData }) => {
  if (!loaderData) return [];
  return getSeoMetaData({ title: loaderData.page.seoTitle });
};
```

Also update `matches[*].data` references to `matches[*].loaderData`.

## Verification workflow

- Check live source, not only browser DOM: crawlers rely on initial HTML.
- Probe representative URLs with `curl`/script and assert presence of:
  - `<title>`
  - `<meta name="description" ...>`
  - `<meta property="og:title" ...>`
  - `<meta property="og:description" ...>`
- Query Sanity production read-only for counts and samples:
  - `count(*[defined(seoTitle) && seoTitle != ""])`
  - `count(*[defined(seoDescription) && seoDescription != ""])`
  - sample `_type,title,slug,seoTitle,seoDescription`
- Inspect git history for dependency bumps around React Router/Remix and CMS packages separately. A Sanity upgrade may be temporally related but not causal if CMS fields/data remain intact.

## Technical canonical and public-origin repairs

1. Separate malformed markup, incorrect public origin and intentional canonical destination policy. Preserve deliberate office-page targets, URL query/hash and existing robots directives. Transaction-specific summaries remain non-indexable; a heading repair does not make them SEO landing pages.
2. Reproduce the descriptor failure through the installed router's actual `Meta`, not a homemade descriptor serializer. In isolated SSR, use `UNSAFE_FrameworkContext` with routeModules metadata alongside a static router and synthetic loaders. Assert exactly one `link[rel=canonical]` and no `meta[rel=canonical]`; check installed descriptor types for `tagName: 'link'` support.
3. Keep server configuration imports out of client-side metadata code. Reuse the existing public-origin rule through a pure resolver and retain a Config-backed server wrapper. Pass route `matches` explicitly into the shared metadata helper; read root `loaderData.env` with narrow unknown guards. Do not mutate loader/business URLs to repair only canonical and `og:url`.
4. Test every affected route's intentional destination and robots policy. Exercise PROD, QA and unknown/local environments; retain path/query/hash exactly while correcting only the selected public origin. Count mechanical call-site edits separately from new logic when explaining PR scope.
5. Hydrate and navigate with real router metadata in a minimal browser fixture. For Vite library fixtures, explicitly define `process.env.NODE_ENV` so output does not reference a missing browser `process`. Label synthetic loaders rather than claiming full-app E2E.
6. For summary headings, render the actual component and styles at mobile/desktop widths. Verify sole H1, subordinate course heading, unchanged typography/margins, no overflow and unchanged continuation controls.

## Reporting pattern

State clearly:

- **Observed:** which live URLs lack source metadata.
- **Not observed:** whether Sanity fields/content are still present.
- **Likely cause:** frontend rendering/regression vs CMS deletion.
- **Fix:** exact mechanical code migration and verification list.

Avoid write actions to CMS/GitHub unless explicitly approved; read-only audit is enough to identify this class of issue.
