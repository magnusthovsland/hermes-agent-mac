# Website PostHog identity and funnel audit

Use when traffic appears inconsistent with People profiles, or when verifying on-site journeys, consent and attribution. Keep the audit inside the user's specified system boundary.

## 1. Verify access and the production target

1. Load the credential without displaying it; call `GET /api/personal_api_keys/@current/` and inspect only label, scopes and `scoped_teams`.
2. Fetch each scoped target with `GET /api/projects/:id/`. A project-scoped key may reject `/api/projects/` with 403 while direct project endpoints and queries work; do not request broader permissions just to enumerate projects.
3. Match the active production capture token to project metadata. A bundle may contain both PROD and QA tokens: inspect the environment branch, not merely whether a token appears anywhere in the file.
4. Keep credentials out of source, reports, command arguments and saved request logs. If access is unavailable, separate source-code findings from unverified ingestion counts.

## 2. Establish a consistent data baseline

Use read-only `POST /api/projects/:id/query/` with `{"query":{"kind":"HogQLQuery","query":"..."}}`. Freeze one UTC start/end for all related queries; rolling `now()` across live queries creates inconsistent totals.

Query these separately:
- Total events, `$pageview` events, distinct `distinct_id`, and nonempty `$session_id` values.
- Event names with event and visitor-ID counts.
- `SELECT count(), min(created_at), max(created_at) FROM persons`.
- Events grouped by `properties.$is_identified` and `properties.$process_person_profile`.
- Hostnames via `properties.$host`, including preview deployments and sibling sites before applying the target-host filter.

Keep results aggregated; never print raw visitor IDs, IPs, click IDs, or customer URLs. Save each query and its aggregate result locally so reported counts can be checked programmatically.

Interpret `person_profiles: identified_only` as permission to capture anonymous events without People records. Few People records do not establish missing tracking, and person profiles are not required for anonymous funnels. Distinguish this explanation from the user's functional requirement: if searchable visitor profiles and per-visitor event history are explicitly wanted, include profile creation in the acceptance criteria. Select a supported profile-creation approach behind analysis consent; do not enable `always` unconditionally merely to populate People, invent customer identities, or assume profiles recover fragmented historical IDs.

## 3. Test continuity rather than assuming it

Group pageviews by `distinct_id` and by `$session_id`; count distinct paths, repeated pageviews, and distinct IDs within a session. Filter empty IDs before counting sessions. Multi-path identities prove some continuity, not universal continuity; a predominance of one-page IDs can reflect bounce behavior, storage fragmentation or other causes.

Inspect the live site's shipped initialization and imports, distinguishing application calls from SDK implementations of `identify`, `set_config` and consent methods. Compare actual Teoria code when using it as a reference.

In a controlled test session, check identity across:
1. First landing with consent undecided.
2. Accepting analysis, rejecting, and withdrawing consent.
3. SPA navigation versus full page reload.
4. The site's own package/course selection and summary steps.

Do not register customers or complete purchases during a read-only audit. HeadlessChrome may be suppressed by SDK bot filters: static/config HTTP 200 responses prove loading, not ingestion. Missing sends from an automation browser alone do not prove production tracking failure.

## 4. Separate consent, persistence and profiles

`persistence: consent ? localStorage+cookie : memory` controls storage, not whether collection is enabled. Inspect whether consent changes update the SDK after initialization; a one-time check may leave a newly consenting visitor in memory mode until reload.

Inspect `cookieless_server_hash_mode` in project metadata and `cookieless_mode` in the client separately. Memory-only persistence is not PostHog server-hash cookieless mode.

When comparing CookieScript implementations, inspect the consent reader and change listeners rather than treating a shared vendor as equivalent behavior. Prefer the loaded instance's `currentState()` when available; safely fall back to the cookie. Test raw and URI-encoded cookie values, categories represented as arrays or JSON-encoded strings, malformed input, missing fields and SDK-not-yet-loaded states. Guard parsing and validate arrays before `includes`; an exception or stale startup-only decision can leave tracking on the wrong storage/consent path. Verify the actual CookieScript event API before wiring callbacks.

For a consented-profile change, test pending → accept, already accepted at load, reject, accept → withdraw, SPA navigation and full reload in QA. Include removing only the performance/analysis category while retaining marketing or functionality; a handler for `CookieScriptReject` alone does not establish correct partial withdrawal. Verify actual profile creation, unchanged ID when appropriate, expected events exactly once and effective collection/storage changes on withdrawal. Do not describe a source comparison or successful API access as an implemented fix.

### Compare a reference implementation in three layers

1. **Source:** Fetch the current target and reference branches through authenticated GitHub reads. Follow imports from entry/root files to the actual SDK initializer, consent utility and identity calls; filenames such as `posthog.config` may contain only tokens/hosts. Search application code for `person_profiles`, `opt_out_capturing_by_default`, `cookieless_mode`, `set_config`, `opt_in_capturing`, `opt_out_capturing`, CookieScript listeners and `identify`. Distinguish repository intent from deployed behavior.
2. **Published CMP:** Extract the exact `cdn.cookie-script.com/s/...js` URL from each live site's HTML and inspect its served configuration. Do not infer that Consent Mode is enabled merely because generic library code contains `ad_user_data` or `analytics_storage`; those branches can be shipped while disabled.
3. **Runtime before a choice:** In a controlled browser session with no prior consent, read `window.CookieScriptData`, `CookieScript.instance.currentState()` and consent entries from `window.dataLayer`. A useful read-only expression is `(window.dataLayer || []).filter(x => x && x[0] === 'consent').map(x => Array.from(x))`. Inspect actual default values and ordering, not only the CMP's self-validation flags. Preserve the observations locally, excluding identifiers and secrets. Missing `window.posthog` is not evidence of missing tracking when the app imports the SDK as a module.
4. **Unverified remainder:** Ask for CMP/GTM read access only for settings or container behavior not established by these checks. State that a public first-load comparison is not a full accept/reject/withdrawal test or a complete tag audit.

Compare Google consent and PostHog separately. For Google, check default/update handling for `analytics_storage`, `ad_storage`, `ad_user_data` and `ad_personalization`, whether the site uses direct `gtag` or GTM, and which component owns consent signals. Do not enable duplicate CMP/GTM consent writers. Treat Basic versus Advanced as an explicit design decision, not an automatic checkbox change. Google Consent Mode does not create PostHog profiles or wire PostHog opt-in/out.

When both implementations use `identified_only`, inspect what actually creates a profile. A reference app may identify an authenticated user only after sign-in; copying its initializer and CMP listeners alone will still not create the requested pre-login visitor profile. Specify a supported consent-gated anonymous profile-creation step separately, without importing the reference app's customer-ID or authentication flow. Likewise, verify both client and project-side configuration before copying a cookieless option.

Enumerate event property names without values using:

```sql
SELECT DISTINCT arrayJoin(JSONExtractKeys(properties)) AS property_name
FROM events
WHERE <fixed time and host filter> AND event = '$pageview'
LIMIT 250
```

Avoid the reserved alias `key`; paginate or narrow the query if the result reaches the limit. Absence of a consent property prevents reliable consent segmentation from those events alone; it does not prove consent was absent.

Do not state either that rejection forbids every form of aggregate measurement or that cookieless tracking is automatically lawful. Consult current official documentation and applicable regulator guidance before proposing no-consent collection. Keep technical capability separate from legal assessment. Distinguish analysis consent from marketing consent; neither should require accepting every category. Persistent/click identifiers and behavioral histories are not necessarily anonymous merely because names and email addresses are absent.

## 5. Build the on-site funnel from the existing event inventory

Find actual package, office, course/date and summary events before inventing replacements. Fetch saved insights and dashboards, then inspect each relevant insight's `query.source.series` and filters. Compare every configured event name—especially the terminal departure event—with the received event inventory for the same fixed period. A stale or missing final event can manufacture apparent abandonment; repair instrumentation or the query before interpreting customer behavior. Check expected office/product/session properties on the relevant events rather than assuming they are populated.

Inspect consent dashboards too: an autocaptured accept/reject button click measures interaction with a banner, not the effective consent categories attached to subsequent events. Use ordered events with an explicit identity basis, time window and product-path conditions. Do not calculate drop-off from unrelated raw totals or combine mutually exclusive paths.

For generic click capture, inspect the top-level `events.elements_chain`, not `properties.$elements_chain`. Verify the field is populated before interpreting zero matches. Example:

```sql
SELECT count(), countIf(not empty(elements_chain)),
       countIf(elements_chain LIKE '%/s/inngang%')
FROM events
WHERE <fixed time and host filter> AND event = '$autocapture'
```

A URL match may include login/registration controls rather than the intended checkout action. Inspect element semantics and event type before labeling it as funnel completion. Prefer a dedicated named departure event if the generic capture is ambiguous.

## 6. Inventory attribution only after the identity baseline

Check existing event properties and URL parameters before claiming attribution is absent. Preserve exact source names: `gclid`, `gbraid`, `wbraid`, `fbclid`, `_fbc`, `_fbp`, `ttclid`, `_ttp`, `ScCid`, `_scid`, `msclkid`, `oppref`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_id`, `utm_content`. Distinguish requested source casing from stored SDK names such as `sccid`.

Report presence, persistence through the on-site journey, consent gating and downstream conversion delivery as separate questions. Finding `gclid` on events proves collection, not valid consent or complete attribution. Verify provider-specific conversion-field mappings against official documentation before implementation.

## 7. Report findings and request only necessary changes

Lead with measured activity and the People-versus-visitors explanation. State which identity behaviors were demonstrated and which remain hypotheses. List existing usable events, host contamination and consent observability gaps. Keep Wright's completion boundary at departure for Ovio; do not combine their projects.

For each proposed change, specify the responsible surface and minimum permission: source repository for SDK/events, consent manager for callbacks/categories, PostHog project settings for project configuration, and insight/dashboard write only to save reports. No write permission is needed for the read-only queries above.
