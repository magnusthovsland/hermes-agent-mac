# Web performance lab audits: Lighthouse CLI, PSI, and framework migrations

Use this reference for Lighthouse/PageSpeed audits and for evaluating migrations from hydrated React-style landing pages to static/islands frameworks such as Astro.

## Never conflate Lighthouse CLI with hosted PageSpeed Insights

Record the measurement source in every table and conclusion:

- **Hosted PSI**: the score shown by `pagespeed.web.dev` or returned by the PageSpeed API.
- **Local Lighthouse CLI**: Lighthouse run from the agent machine, even when `--form-factor=mobile --throttling-method=simulate` is used.
- **Field data**: CrUX, RUM, or PostHog measurements; do not compare these as if they were lab scores.

A local simulated mobile run is not a PSI run. Its trace begins on the local host, browser version, CPU, network route, CDN node, cache state, and third-party timing. PSI uses Google-hosted infrastructure. Two runs can therefore have materially different FCP/LCP and scores while using similar Lighthouse throttling settings.

### Required reporting shape

Always report separately:

```text
Hosted PageSpeed Insights mobile: <score or unavailable>
Local Lighthouse CLI mobile: <score(s)>
Field/RUM: <metric and percentile>
```

If the user's business concern is Google Ads or the hosted PSI result, use the hosted PSI score as the governing lab baseline. Local CLI traces remain valuable for element attribution, request breakdowns, long tasks, bundle analysis, and repeatable diagnostics, but must not replace or be labelled as PSI.

Capture for every CLI run:

- Lighthouse version and fetch time
- host and network user agents
- form factor, viewport, DPR
- throttling method and values
- at least two runs when score variance matters
- LCP element and metric values

If the PageSpeed API is quota-limited, say so and use the user's hosted result rather than implying the local result supersedes it.

## Production wins over local source

When the production bundle, HTML, or headers differ from a local checkout:

1. Treat production behavior as authoritative for the audit.
2. Label local file/line evidence as potentially stale.
3. Cite live asset names, headers, DOM, and Lighthouse nodes for production claims.
4. Do not infer that a local branch represents the deployed build merely because `git status` is clean.

## Framework migration evaluation

A static/islands framework can remove app-controlled costs, but does not automatically fix third-party costs.

Classify each finding as:

1. **Solved by architecture** — e.g. no hydration for static content, no framework runtime for server-only components.
2. **Made easier** — e.g. route-scoped CSS, explicit script ordering, responsive image generation.
3. **Unchanged unless explicitly redesigned** — e.g. gtag, CMP, PostHog recorder/surveys, upstream cache TTLs, consent and attribution.

For Astro-style migrations, distinguish:

- **Astro-native static page with a few islands**: likely large JS/hydration reduction.
- **Astro shell around one large `client:load` React tree**: likely small improvement.

Map the existing page into static sections and true interactive islands. Recommend `client:visible` or `client:idle` only where interaction requires hydration. Do not assume image optimization happens when remote URLs are placed in raw `<img>` tags; verify the image service, remote-domain configuration, generated `srcset`, `sizes`, dimensions, quality, and formats.

## Prediction discipline

Performance score forecasts are estimates, not measurements. Base them on the correct baseline and provide scenarios rather than one precise promise:

- current hosted PSI
- local CLI range
- framework-only migration
- framework plus analytics/font/image redesign

Do not project a hosted PSI score from local Lighthouse scores without clearly labelling the uncertainty.

## Auditing an in-progress Astro migration from compiled output

When source-repository access is unavailable but a public preview deployment exists, do not stop at the framework name. Treat the compiled deployment as evidence and explicitly separate **confirmed runtime behavior** from **source/config inference**.

Inspect:

1. Generated HTML, response headers, `Last-Modified`, CDN cache state, canonical/hreflang/schema, and route coverage.
2. `<astro-island>` elements: component URL, renderer URL, serialized props, SSR marker, and hydration directive (`idle`, `visible`, `load`, etc.).
3. Actual transferred sizes for the framework renderer, React islands, shared component libraries, CSS, fonts, and third parties.
4. Whether below-fold images/components are genuinely absent from the initial request set or merely hidden.
5. Standalone preview routing versus intended production composition: root/legal/action URLs may intentionally remain in an older app, so label them as broken only in the standalone preview until domain routing is verified.
6. Hashed `/_astro/*` asset cache headers; a hashed filename with `max-age=0, must-revalidate` is a deployment/config weakness even when first-load metrics are good.

### React-to-Astro inline-script trap

React/JSX patterns such as:

```astro
<script>
  {`
    posthog.init(...)
  `}
</script>
```

can compile into a syntactically valid JavaScript block containing an inert template-string expression. The intended code does **not execute**, and the console may show no syntax error. This can silently disable PostHog, consent defaults, GCLID propagation, or conversion tracking while making Lighthouse look better.

Verification must therefore include runtime state, not source appearance alone:

- inspect the emitted `<script>` text;
- check expected globals (`window.posthog`, `dataLayer`, etc.);
- inspect the order of queued consent/config commands;
- confirm expected network requests and events;
- test attribution fields through the full CTA flow when writes are permitted.

Do not credit a score improvement caused by a broken analytics integration. Report architecture gains separately from missing functionality.

### `content-visibility` and deferred-island caveats

`content-visibility: auto`, lazy images, and `client:visible` can legitimately remove below-fold work from initial load. However:

- a full-page screenshot may show blank intrinsic-size placeholders because offscreen content was never painted;
- a fixed `contain-intrinsic-size` used for unlike sections can distort scroll geometry or produce shifts when content becomes visible;
- SSR HTML and serialized island props can duplicate large FAQ/product payloads;
- a visible-only accordion may briefly be non-interactive before hydration.

Test normal scrolling, mobile viewport behavior, scroll-position stability, accessibility, and post-scroll resource loading. Prefer native HTML such as `<details>/<summary>` when it can eliminate an otherwise large React/Mantine island.

### Fair migration comparisons

For before/after tables, keep the same measurement class and disclose functional differences. In particular, note whether the preview:

- delays gtag beyond the critical render path;
- does not initialize PostHog;
- never loads below-fold images or a `client:visible` island during the trace;
- uses a different deployment/CDN route.

A 99 local Lighthouse result can be valid for the observed initial-load architecture and still be an invalid production forecast if analytics or conversion behavior is absent.

## Auditing analytics regressions after broken scripts are restored

When a previously inert analytics integration is fixed, compare like with like and let the user’s repeated **hosted PSI** before/after baseline govern. Never use faster local Lighthouse CLI runs to dismiss a stable hosted regression.

1. Verify runtime functionality first: expected global is loaded, requests fire, extensions initialize, and consent ordering is correct.
2. Confirm functional parity. A high hosted PSI baseline with inert analytics is not a valid full-production target, but it is a useful `analytics absent` control.
3. Separate tiny ordering changes from newly activated payload. Moving Consent Mode default ahead of `gtag config` adds negligible work; repairing an inert PostHog bootstrap can activate the SDK, recorder, surveys, dead-click capture, exceptions, web-vitals, flags, remote config, and API calls.
4. Use at least 3–5 repeated **hosted PSI** runs per variant and report median/range. Local CLI remains diagnostic for element attribution and payload analysis, not a substitute baseline.
5. Identify the LCP node and compare FCP, LCP, TBT, CLS, and Speed Index. A stable hosted pattern such as `FCP ≈ 1.2 s`, `TBT ≈ 10–20 ms`, `CLS = 0`, but `LCP ≈ 4.4 s` is still a real LCP/render-completion regression. Async analytics can compete for bandwidth, initialize observers, or alter repaint timing without producing high TBT.
6. Quantify analytics separately: request count, compressed/decompressed bytes, extension modules, third-party main-thread time, boot-up time, and unused JavaScript.
7. Run matched preview variants: full analytics, analytics absent, lightweight analytics, and deferred analytics. Keep consent ordering identical. A blocked-domain test is an imperfect but useful control because failures/retries can alter scheduling.
8. Calibrate causality to the whole evidence set. If hosted PSI was repeatedly ~98 with inert analytics, repeatedly ~83 after full analytics became active, and an analytics-absent control returns ~97–98, treat analytics activation as the primary suspect. Do not blame the consent-order correction without isolated evidence.

For PostHog-style integrations, inspect whether the base SDK immediately pulls session recording, surveys, dead-click capture, web-vitals, exception capture, feature flags, and remote config. Prefer preserving explicit pageview/CTA/purchase events while disabling or deferring unused extensions. Also distinguish memory-only persistence from no collection: a tool may send network events before consent even when it does not yet write durable cookies/local storage. Treat that as an explicit privacy/product decision, not merely a performance toggle.

Finally, distinguish hosted PSI from Google Ads Landing Page Experience: PSI is strong technical evidence, but Google does not document a one-to-one visible-score-to-LPE formula.

## Text LCP and Astro/islands payload checks

The LCP element may be a large paragraph rather than an image. Record its exact node/selector and examine font loading, render-blocking CSS, layout, transitions, and third-party timing before recommending image optimization.

For Astro production audits, additionally quantify:

- decoded and transferred HTML size;
- repeated content across SSR markup, serialized island props, and JSON-LD;
- each island’s hydration directive and generated markup size;
- React/Mantine renderer and component bundle sizes;
- whether native `<details>/<summary>` or a small vanilla-JS island can replace a hydrated FAQ/header;
- font cache headers (prefer versioned immutable assets for static fonts);
- hashed asset cache headers and render-blocking CSS.

A useful architecture can still ship oversized HTML and framework bundles: “only two islands” is not enough without measuring those islands.

## Three different Google performance timelines

Always clarify which “Google landing-page number” the user means:

1. **Lighthouse/PSI lab score:** recomputed on every run and can vary immediately; use repeated runs and a median.
2. **CrUX/PageSpeed field data:** rolling 28-day real-user window, so deployment effects enter gradually.
3. **Google Ads Landing page experience:** keyword-level Above/Average/Below diagnostic, not a 0–100 Lighthouse score. Google does not publish a guaranteed refresh interval; avoid promising a daily update.