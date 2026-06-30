# React Router v8 route metadata regression

Use when a React Router/Remix-style frontend loses `<title>`, `meta name="description"`, Open Graph, or Twitter metadata after a framework upgrade.

## Symptom

- Live HTML/source contains route content but lacks page-specific metadata.
- Root/default metadata may still appear on routes without dynamic `meta()`.
- CMS data and SEO fields can still exist; the failure is in route metadata rendering.

## Durable cause to check

React Router v8 removed the deprecated `data` field passed to route module `meta()` functions. Code written for v7 often has:

```ts
export const meta: MetaFunction<typeof loader> = ({ data }) => {
  if (!data) return [];
  return [{ title: data.page.seoTitle }];
};
```

In v8, use `loaderData`:

```ts
export const meta: MetaFunction<typeof loader> = ({ loaderData }) => {
  if (!loaderData) return [];
  return [{ title: loaderData.page.seoTitle }];
};
```

Also update `data?.foo` to `loaderData?.foo`. Search only within route `meta()` functions so component-level `data` variables are not accidentally changed.

## Investigation pattern

1. Verify live HTML with source-level checks, not only browser DOM after hydration:
   - `<title>`
   - `<meta name="description">`
   - `<meta property="og:title">`
   - `<meta property="og:description">`
2. Verify CMS data separately before assuming content deletion:
   - Count documents with `seoTitle` / `seoDescription`.
   - Sample values from production/QA read-only.
3. Check dependency/git history for React Router v7→v8 or Remix→React Router framework upgrades.
4. Search route modules for stale meta signatures:

```bash
rg "export const meta: MetaFunction<[^=]+> = \(\{ data" frontend/app/routes
```

5. Patch `meta()` functions to use `loaderData` and rerun build/typecheck.

## Verification

- `npm run build` should complete.
- `npm run typecheck` may reveal unrelated pre-existing project type debt; distinguish those from errors introduced by the metadata patch.
- After deployment, view source for representative URLs and confirm route-specific metadata is rendered server-side.