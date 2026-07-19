# Google Ads landing-page experience diagnostic notes

Use these notes for PPC landing-page audits where Google Ads shows Quality Score capped by `Landing page exp.: Below average` even when `Exp. CTR` and `Ad relevance` are above average.

## Audit sequence

1. **Confirm domain/product and exact final URL.** Do not infer from adjacent product memory. Ask or inspect the screenshot/context if multiple brands are plausible.
2. **Fetch candidate URLs with AdsBot user agents.** At minimum:
   - `AdsBot-Google-Mobile (+http://www.google.com/mobile/adsbot.html)`
   - `AdsBot-Google (+http://www.google.com/adsbot.html)`
   - Googlebot smartphone UA
3. **Check URL resolution and fallbacks.** For keyword-like paths, record status, final URL, redirect history, title, description, H1 and word counts. A `200` generic homepage for `/gratis-teoriprove-bil`-style URLs is a soft-404/fallback risk and weakens message match.
4. **Inspect sitemap for the intended landing page.** Campaigns should usually point to the most specific page, not the homepage.
5. **Compare keyword-message match.** Build a table for term counts and above-the-fold copy:
   - keyword phrase
   - title/meta description
   - H1
   - intro/subheading
   - CTA
   - proof/reassurance near CTA
6. **Run mobile performance audit.** Use PageSpeed API if available; otherwise local Lighthouse. Capture FCP, LCP, CLS, TBT, performance score, and top unused/heavy JS.
7. **Separate causes.** Google Ads LPE problems typically fall into:
   - wrong/generic final URL,
   - weak above-the-fold message match,
   - poor mobile speed/Core Web Vitals,
   - intrusive or heavy third-party scripts,
   - crawl/indexability/canonical issues.

## Teoria.no pattern observed

For keywords around `gratis teoriprøve bil`, the specific page was:

```text
https://teoria.no/teoriprove/bil/gratis
```

It had strong basics:

```text
Title: Øv gratis til teoriprøven for bil | Teoria
H1: Øv gratis til teoriprøven for bil
SEO/accessibility/best-practices Lighthouse: 100
```

But several obvious keyword URL variants returned `200` and fell back to the homepage:

```text
/gratis-teoriprove
/teoriprove
/teoriprove-bil
/gratis-teoriprove-bil
/klasse-b
```

This is a durable PPC pitfall: if Google Ads final URLs, sitelinks, tracking-expanded URLs, or old URLs land on a generic homepage instead of the specific page, `Landing page experience` can remain below average even when CTR/ad relevance are high.

## Recommended fixes for this pattern

- Ensure all ad group keywords for `gratis teoriprøve bil`, `teoriprøve gratis`, and `klasse B` land on the specific page.
- Add 301 redirects from simple keyword URLs to the specific page, e.g. `/gratis-teoriprove-bil -> /teoriprove/bil/gratis`.
- Do not return `200` homepage for unknown/keyword-like paths; use relevant redirects or true 404.
- Strengthen above-the-fold exact match:
  - H1: `Gratis teoriprøve for bil – klasse B`
  - Intro: mention `førerkort klasse B`, realistic tests, no card/no binding, and free trial duration.
  - CTA: `Start gratis teoriprøve` or similar.
- Add FAQ entries matching paid-search questions: `Er teoriprøven for bil gratis?`, `Kan jeg øve til teoriprøven klasse B gratis?`, `Må jeg legge inn bankkort?`.
- Defer non-critical third-party JS (analytics recorder/surveys, heavy cookie scripts) until consent/interaction where possible.

## When Google Ads account/API access exists

Do not stop at website-only hypotheses if the user says Google Ads access is available. Pull live account data first, then rank causes.

Minimum read-only Google Ads API checks:

1. **Campaign/account overview** for the relevant customer and date range: status, budgets, bidding strategy, impressions, clicks, CTR, cost, CPC, conversions.
2. **Keyword Quality Score components** from `keyword_view`:
   - `quality_info.quality_score`
   - `quality_info.post_click_quality_score` (Landing page experience)
   - `quality_info.creative_quality_score` (Ad relevance)
   - `quality_info.search_predicted_ctr` (Expected CTR)
3. **Final/expanded landing URLs** from `expanded_landing_page_view` and ad final URLs from `ad_group_ad` before blaming redirects or wrong pages.
4. **Search terms** by cost/clicks/conversions to identify intent mismatch and negative-keyword opportunities.
5. **Conversion actions and goals** (`conversion_action`, `customer_conversion_goal`, `campaign_conversion_goal`) to detect inconsistent biddable goals or primary/include settings.
6. **Live URL checks** for the highest-click expanded final URLs using AdsBot user agents.
7. **Mobile Lighthouse/PageSpeed** for the actual highest-click final URL(s), not just guessed pages.

Interpretation pattern:
- If `Expected CTR` and `Ad relevance` are above/average but `post_click_quality_score` is almost entirely `BELOW_AVERAGE`, the cap is likely landing-page/message/speed rather than ad copy.
- If final URLs are already specific and correct, do not overstate URL mismatch. Focus on above-the-fold message match, mobile LCP/FCP, intrusive/heavy scripts, and conversion-goal consistency.
- Report weighted Quality Score and Landing Page Experience by impressions, not just a few visible keywords.

## Reporting format

Use a concise structure:

1. **Likely root causes** ranked by confidence.
2. **Evidence** from live Google Ads data, live URL fetches, and Lighthouse.
3. **Ads account checks**: final URL, mobile final URL, tracking template, final URL suffix, sitelinks, keyword QS components, search terms, conversion goals.
4. **Landing-page changes**: copy/message match, redirects, technical speed.
5. **Expected timeline**: Google Ads may need days to a few weeks to recrawl/re-score.

## Refresh cadence and metric confusion

Do not answer “how often does Google update the landing-page number?” until the metric is identified:

- A Lighthouse/PageSpeed **lab score (0–100)** is recomputed on each run and can change immediately.
- PageSpeed **CrUX field metrics** summarize a rolling 28-day real-user window, so a deployment’s impact appears gradually.
- Google Ads **Landing page experience** is a keyword-level `Above average` / `Average` / `Below average` Quality Score component, not a Lighthouse score.

Google’s official Quality Score documentation says component status compares advertisers for the same exact search over the previous 90 days, but it does not promise a fixed daily refresh cadence. State that no guaranteed interval is published. Give “days to a few weeks” only as a practical expectation, clearly labelled as such, and expect longer delays when eligible exact-match traffic is sparse.

Never infer that an immediate Lighthouse improvement will cause an immediate Google Ads status change. When evaluating a recent deployment, preserve the deployment date and monitor both current and historical Quality Score component columns alongside impressions.