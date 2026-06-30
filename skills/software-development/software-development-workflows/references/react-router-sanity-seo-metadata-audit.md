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

## Reporting pattern

State clearly:

- **Observed:** which live URLs lack source metadata.
- **Not observed:** whether Sanity fields/content are still present.
- **Likely cause:** frontend rendering/regression vs CMS deletion.
- **Fix:** exact mechanical code migration and verification list.

Avoid write actions to CMS/GitHub unless explicitly approved; read-only audit is enough to identify this class of issue.
