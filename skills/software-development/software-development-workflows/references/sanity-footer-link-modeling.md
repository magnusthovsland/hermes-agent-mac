# Sanity footer/navigation link modeling

Use this when a Sanity-managed footer/header/nav needs to link to a route that exists in the frontend but is not represented as a Sanity page document.

## Problem pattern

A CMS editor is managing a footer link list in Sanity. Existing link object types may include:

- `linkInternal`: reference to Sanity page/document types, resolved with `getPathFromSanityPage(...)` or equivalent.
- `linkExternal`: URL field, often already supported for header/footer arrays.
- `linkRelative`: sometimes exists for app routes such as logged-in pages, but may not be enabled for footer/header link arrays.

The visible symptom is a Sanity Studio modal requiring an `Intern lenke` reference for a page like `/gavekort`, even though the route exists only in the frontend and has no Sanity document.

## Recommended decision order

1. **Inspect schema first, read-only.** Check the link-list schema and frontend renderer before recommending content workarounds.
   - CMS schema examples: `cms/schemas/components/linkList.ts`, `linkInternal.ts`, `linkExternal.ts`, `linkRelative.ts`.
   - Frontend examples: footer/header components and GROQ singleton queries that project the link fields.
2. **Do not create fake Sanity pages just to satisfy an internal reference.** This creates misleading content structure and future cleanup work.
3. **If `linkExternal` is already supported**, use it as the short-term workaround for same-domain non-Sanity routes:
   - title: `Gavekort`
   - url: `https://www.wright.no/gavekort`
   Treat it as external from Sanity's perspective even if it is same-site.
4. **If Studio does not show the external-link option despite schema support**, suspect the deployed Studio build/schema is stale. Recommend redeploying Sanity Studio rather than changing content data.
5. **For the durable model, add/enable a relative link type** in the footer/header arrays for frontend-only same-site routes:
   - `_type: linkRelative`
   - `relativeUrl: /gavekort`
   Frontend rendering should handle `linkRelative` separately from Sanity references and full external URLs.

## Content-editor guidance

If an existing footer item was created as `linkInternal`, the editor normally cannot turn that object into `linkExternal` in place. The practical UI fix is to delete that link item and add a new item with the correct object type.

## Reporting format for users

Separate:

- **Immediate operational workaround**: what the editor can do in Sanity now.
- **Likely technical cause**: stale Studio build vs schema/model limitation.
- **Long-term clean model**: internal Sanity reference vs relative frontend route vs external URL.

Avoid presenting a code change as necessary until the repo/schema has been checked; many projects already support `linkExternal` but the editor has selected the wrong object type or prod Studio is stale.
