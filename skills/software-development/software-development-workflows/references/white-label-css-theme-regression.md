# White-label CSS theme regression investigation

Use this playbook when several tenant sites suddenly lose brand colors and colored cards become white or transparent.

## Core failure pattern

Many white-label frontends scope design tokens under a tenant class:

```css
.tenant-a { --dominant-12: #123456; --complimentary-4: #eef3ff; }
.bg-dominant { background-color: var(--dominant-12); }
```

If the tenant class disappears from `<html>` or `<body>`, the stylesheet can still load successfully while every dependent declaration becomes invalid. Browsers then compute transparent backgrounds, missing shadows, and fallback text/border colors. This is not the same as a missing CSS bundle.

A common trigger is upstream data returning `colorTheme: ""`. Nullish fallback does not catch blank strings:

```ts
"" ?? "fallback" // ""
```

## Read-only diagnostic sequence

1. Reproduce every affected domain visually. Record whether content/layout/fonts load while brand fills disappear.
2. Check console errors, but do not stop when there are none; undefined CSS custom properties are usually silent.
3. Confirm stylesheet URLs return 200 and inspect the deployed CSS for:
   - tenant selectors such as `.tenant-a`
   - token definitions such as `--dominant-*`
   - utility rules that reference those tokens
4. Inspect live DOM and computed styles:
   - `<html>` / `<body>` classes
   - expected tenant class count
   - computed token values
   - computed backgrounds on representative elements
5. Inspect server-rendered loader data, not only client state. For React Router streaming HTML, the `streamController.enqueue(...)` payload uses an indexed value table; decode it to recover loader fields such as tenant ID, name, and `colorTheme`.
6. Prove causality with a reversible browser-only test: temporarily add the expected tenant class, read computed tokens/colors, then remove it immediately. Never persist the change.
7. Compare all affected tenants. Blank values across several tenant IDs strongly suggest a shared backend DTO, migration, CMS publish path, or bulk update rather than independent page edits.
8. Correlate deployment timing with Git history, but inspect the actual diff before assigning blame. A deployment can coincide with an upstream data regression while leaving theme code untouched.
9. Verify the repository remained clean and report that no persistent changes were made.

## Evidence interpretation

Strong evidence for an upstream theme-data regression:

- CSS bundle loads and contains correct tenant palettes.
- No runtime JS error explains the visual loss.
- `<html>` lacks the tenant class.
- CSS variables are empty and representative backgrounds compute to `rgba(0, 0, 0, 0)`.
- Server-rendered loader data explicitly contains a blank theme value.
- Temporarily adding the tenant class restores expected colors.

## Repair recommendations

### Immediate restoration
Restore the canonical theme slug in the backend/CMS source of truth. If HTML is dynamically rendered with no CDN caching, this may restore production without a frontend deployment; still verify the live DOM and computed colors.

### Frontend hardening
Treat blank, whitespace-only, and unknown themes as invalid. Do not fall back to a human display name unless it is guaranteed to equal a CSS selector.

```ts
const candidate = rawTheme?.trim().toLowerCase();
const theme = candidate && validThemes.has(candidate)
  ? candidate
  : fallbackThemeByTenantId[tenantId];
```

### Backend/CMS hardening
- Reject blank theme values at publish/update time.
- Distinguish omitted fields from explicit clearing in PATCH/PUT logic.
- Preserve existing values when optional fields are absent.
- Validate against theme slugs shipped by the frontend.
- Audit and log bulk tenant updates and migrations.

### Regression tests
Cover valid, null, undefined, blank, whitespace, and unknown themes; assert the SSR `<html>` class; and add browser checks that key backgrounds are non-transparent on representative tenants.

## Reporting standard
Separate:
- confirmed live behavior,
- confirmed technical mechanism,
- likely introduction point,
- unverified upstream hypotheses,
- immediate restoration,
- long-term hardening.

Do not claim a precise backend/CMS commit without repository or audit-log evidence.