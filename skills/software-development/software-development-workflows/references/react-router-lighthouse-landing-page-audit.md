# React Router landing-page Lighthouse audit

Use this read-only workflow for React Router/Vite/Vercel marketing pages where lab Lighthouse is poor but field data is healthy.

## Measurement order

1. Run Lighthouse mobile at least twice. Lab scores vary; report the range and preserve each JSON result.
2. Identify the LCP from the Lighthouse node/selector (`lcp-breakdown-insight`), not from visual assumptions. A large inline SVG may not be LCP; descriptive text can be.
3. Fetch the actual SSR/prerendered HTML and response headers. Production output wins over local source.
4. Parse the initial `<head>` in emitted order: inline scripts, external scripts, font/image preloads, modulepreloads, and stylesheets.
5. Measure five TTFBs and report the median plus `Cache-Control`, `Age`, and `X-Vercel-Cache`.
6. Read route/framework config to distinguish request-time SSR from prerender/CDN caching.
7. Group transferred JS into:
   - initial app modulepreloads,
   - route/lazy follow-up chunks,
   - analytics/CMP,
   - proxied analytics extensions.
8. Attribute unused JS, execution time, and long tasks by URL. Proxied PostHog assets look first-party by hostname but remain third-party product code.
9. Check repository status before and after. Keep Lighthouse artifacts and downloaded responses under `/tmp` for audit-only work.

## Image audit

- Inspect every rendered `<img>` for `src`, `srcset`, `sizes`, dimensions, `loading`, and `fetchpriority`.
- Do not recommend LCP preload or `fetchpriority=high` when the hero is inline SVG or the measured LCP is text.
- Density `1x/2x` URLs based only on height can still over-fetch. Measure the displayed CSS width, calculate the needed DPR width, request an explicit candidate (for example `w=570&q=70`), and compare actual bytes.
- Verify format support with a real HTTP response. Do not assume a CDN accepts `fm=avif`; some image pipelines return HTTP 400.
- Missing width/height is a robustness issue, but do not call it a performance priority when measured CLS is already near zero.

## CSS, fonts, and script ordering

- Report both compressed transfer size and uncompressed CSS size. Tie savings to Lighthouse's named audit estimate.
- `font-display: swap` does not make font caching irrelevant. A preload followed by a second 304 revalidation can expose `max-age=0`; version the font URL before recommending one-year immutable caching.
- `async`/`defer` scripts may not be parser-blocking but still compete for bandwidth and produce long tasks.
- React resource hoisting can make emitted script order differ from JSX order. If consent default must precede gtag, verify `view-source`/actual HTML, not comments or component order.
- Do not recommend delaying a CMP unless every tracker is hard-gated; otherwise the performance change can increase consent/compliance risk.

## Interpretation pitfalls

- A current production deploy may already contain optimizations absent from a stale local checkout. Flag discrepancies and cite live asset names/headers where exact source lines are unavailable.
- `Use efficient cache lifetimes` can be entirely driven by CookieScript/PostHog rather than the app's hashed assets. Separate controllable app caching from vendor/proxy TTLs.
- A modern Vite target (for example ES2022) can coexist with a Legacy JavaScript finding caused by dependencies or third-party scripts. Do not propose Browserslist changes unless own bundles are responsible.
- Code splitting is not the same as viewport deferral: `React.lazy` components rendered immediately still begin downloading during initial hydration.
- If TBT and CLS are green, say explicitly that score loss is FCP/LCP-driven even when hydration consumes substantial total CPU.

## Reporting format

Lead with a short in-chat conclusion that is readable without opening a file. Then provide:

1. current production measurement range,
2. summary table with status/metric/effort/impact,
3. evidence by requested audit item,
4. prioritized fixes ordered by impact divided by effort,
5. predicted score range with assumptions,
6. explicit contradictions to the initial hypothesis,
7. read-only verification (`git status`/empty diff).

Distinguish measured facts, inferred causes, and recommendations. Keep the first screen concise even when the full report is long.