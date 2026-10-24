# Website PostHog identity and handoff audit

## GTM desktop QA workflow

When the signed-in GTM session is in the user's native Chrome rather than the automation browser, use the authorized Apple Events JavaScript bridge for scoped DOM inspection. Re-discover tab IDs after user tab changes; do not assume headless and native profiles share login. Encode AppleScript string arguments with `json.dumps(..., ensure_ascii=False)` so Norwegian UI labels do not become invalid AppleScript Unicode escapes.

In the newer GTM UI a container with one workspace may hide the workspace selector. Open Overview, use the more-options control inside `gtm-overview-workspace-header` (not the Add Tag dropdown), choose the workspace selector, and click its add icon. Create a dedicated unpublished QA workspace, then verify its name and workspace URL. In pickers, click the actual table/trigger row; clicking a text span can have no effect. After saving a tag, wait for completion and read it back from the tag list/editor before claiming success. Do not equate a saved draft with a published or runtime-tested configuration. If Preview does not open through DOM automation, ask the user to click Preview instead of publishing to make the test work.


Use when traffic appears inconsistent with People profiles, or when verifying on-site journeys, consent and attribution. Keep the audit inside the user's specified system boundary.

## Wright environment pointers

- PostHog EU: Wright.no PROD project `49059`; Wright.no QA project `50123`. Read credential: `~/.hermes/credentials/posthog_wright.json`. Never print its contents.
- GitHub: `WrightTrafikkskole/Wright-Web` (`frontend/`). Reference repos: `teoria-frontend` and `teoria-open-pages` in the same organization. Temporary GitHub credential: `~/.hermes/credentials/github_wright_temporary_token`; validate before use and arrange revocation after the work. Do not assume it remains valid.
- Verified live GTM mapping: `qa.wright.no` → `GTM-MDCP8Q8`; `wright.no` → `GTM-NTMPCTS2`. QA contains experimental configuration; do not treat it as a production mirror.

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

`persistence: consent ? localStorage+cookie : memory` controls storage, not whether collection is enabled. Inspect both application listeners and the CMP's reload-after-accept/reject setting before diagnosing a stale startup-only decision. An automatic full reload can cause the initializer to reread the new consent even without listeners; test whether the ID survives that transition. Do not disable the reload until replacement consent-change handling has passed tests. Reloading does not itself create a person profile.

Inspect `cookieless_server_hash_mode` in project metadata and `cookieless_mode` in the client separately. Memory-only persistence is not PostHog server-hash cookieless mode.

When comparing CookieScript implementations, inspect the consent reader and change listeners rather than treating a shared vendor as equivalent behavior. Prefer the loaded instance's `currentState()` when available; safely fall back to the cookie. Test raw and URI-encoded cookie values, categories represented as arrays or JSON-encoded strings, malformed input, missing fields and SDK-not-yet-loaded states. Guard parsing and validate arrays before `includes`; an exception or stale startup-only decision can leave tracking on the wrong storage/consent path. Verify the actual CookieScript event API before wiring callbacks.

For a consented-profile change, test pending → accept, already accepted at load, reject, accept → withdraw, SPA navigation and full reload in QA. Include removing only the performance/analysis category while retaining marketing or functionality; a handler for `CookieScriptReject` alone does not establish correct partial withdrawal. Verify actual profile creation, unchanged ID when appropriate, expected events exactly once and effective collection/storage changes on withdrawal. Do not describe a source comparison or successful API access as an implemented fix.

### Compare source, published configuration and runtime

1. **Source:** Fetch the current target and reference branches through authenticated GitHub reads. Follow imports from entry/root files to the actual SDK initializer, consent utility and identity calls; filenames such as `posthog.config` may contain only tokens/hosts. Search application code for `person_profiles`, `opt_out_capturing_by_default`, `cookieless_mode`, `set_config`, `opt_in_capturing`, `opt_out_capturing`, CookieScript listeners and `identify`. Distinguish repository intent from deployed behavior.
2. **Published CMP:** Extract the exact `cdn.cookie-script.com/s/...js` URL from each live site's HTML and inspect its served configuration. Do not infer that Consent Mode is enabled merely because generic library code contains `ad_user_data` or `analytics_storage`; those branches can be shipped while disabled.
3. **Runtime before a choice:** In a controlled browser session with no prior consent, read `window.CookieScriptData`, `CookieScript.instance.currentState()` and consent entries from `window.dataLayer`. A useful read-only expression is `(window.dataLayer || []).filter(x => x && x[0] === 'consent').map(x => Array.from(x))`. Treat this as a partial observation: GTM templates can call `setDefaultConsentState` and `updateConsentState` directly without producing `gtag('consent', ...)` entries. Verify effective defaults, updates and ordering in Tag Assistant's Consent view; neither absent dataLayer commands nor CMP self-validation flags establish failure or success alone. Preserve observations locally, excluding identifiers and secrets. Missing `window.posthog` is not evidence of missing tracking when the app imports the SDK as a module.
4. **Unverified remainder:** Ask for CMP/GTM read access only for settings or container behavior not established by these checks. State that a public first-load comparison is not a full accept/reject/withdrawal test or a complete tag audit.

Compare Google consent and PostHog separately. For Google, check default/update handling for `analytics_storage`, `ad_storage`, `ad_user_data` and `ad_personalization`, whether the site uses direct `gtag` or GTM, and which component owns consent signals. Do not enable duplicate CMP/GTM consent writers. Treat Basic versus Advanced as an explicit design decision, not an automatic checkbox change. Google Consent Mode does not create PostHog profiles or wire PostHog opt-in/out.

Standardize consent behavior rather than insisting on identical installation mechanics across Teoria and Wright. Direct CookieScript installation can coexist with GTM distributing permitted tags; GTM installation is also a supported, vendor-recommended approach. Keep one CMP installation path and a clear owner of default/update signals. Build and verify replacement handling before disabling the current path. Distinguish browser GTM (scripts send requests to configured destinations) from server-side GTM or another collection endpoint; using direct CookieScript does not remove the option of server-side routing. Consent must be propagated and enforced independently of routing, including for non-Google tags. Do not infer that Wright has server-side GTM merely from a web container or template screenshot.

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

### Inspect GTM screenshots and template logic in dependency order

1. Confirm the container ID from each live environment's HTML/runtime configuration. Separate QA from PROD and workspace edits from published versions; zero workspace changes does not establish runtime correctness. A successful QA test does not validate a different production container.
2. Read the tag list, then the relevant tag configuration, its referenced variables, and finally the variable/template code. Distinguish firing triggers from exceptions and paused tags. A `CookieScriptGoogleConsentUpdated` trigger signals an update, not necessarily granted consent; inspect each tag's effective consent requirements.
3. Distinguish the **CookieScript CMP tag template** from the **CookieScript Consent State variable template**. The former owns tag initialization and consent updates; the latter parses the cookie used by CJS category checks. A permission to read `CookieScriptConsent` proves only capability, not parsing behavior. Request the correct variable template's Code tab or Preview return value rather than repeatedly asking for permission-panel screenshots.
4. Trace whether the website and GTM both reference the same CMP script. Treat matching URLs as a possible overlap, not proof of double execution; inspect the published loading/guard behavior before removing either source. Likewise, a disabled Consent Mode switch in the CMP dashboard does not rule out a GTM-based integration.
5. Inspect the exact installed template before acting on an available-update banner. An update notice is not proof of a defect; compare behavior and test before publishing changes.

### Validate CookieScript category parsers with GTM semantics

Read the full variable code before asserting that `.includes()` fails on a first visit. If the parser initializes its return value to `''` and returns that when the cookie is missing, the downstream string `.includes()` calls safely return false.

Test the exact installed parser against these fixtures:
- Missing cookie and denied read permission: fail closed with an empty category value.
- Explicit rejection: do not grant categories.
- Accepted consent with categories as a JSON-encoded string: parse once into a list.
- Accepted consent with categories already an array: use the array, do not JSON-parse it again.
- Missing categories, malformed JSON, invalid URI encoding, nulls and wrong types: return a safe empty value without exceptions or implicit approval.
- Partial analysis/marketing consent: return only the selected categories.

Model GTM's sandbox `require('JSON').parse` accurately in local tests: it coerces non-string inputs to strings and returns `undefined` on invalid JSON, rather than throwing like ordinary JavaScript `JSON.parse`. A local adapter can use `try { return JSON.parse(String(value)); } catch { return undefined; }`; label this as an emulated unit test, not actual GTM execution. An already-parsed category array is coerced to a comma-separated string, commonly yielding `undefined`; calling `.join()` on that result fails. Malformed outer JSON can similarly fail at `.action` unless the parsed object is validated. Check the sandbox API documentation before relying on browser-language assumptions.

Do not treat a bare return value of `'accept'` as category approval when downstream code tests for `'performance'` or `'targeting'`; those checks still return false. Compare parser support with the live cookie format before claiming affected traffic volume. Validate the repaired parser in template tests and Preview, then inspect effective consent defaults/updates for no choice, analysis-only, all, rejection and partial withdrawal before publishing.

Keep unrelated ecommerce findings separate: when a GTM price variable divides by 100, inspect the actual source dataLayer units and tag references first. A source value already converted from minor units can be divided twice, but unused variables do not affect production. Check undefined references and in-place item mutation without treating a screenshot as proof that a tag executed.

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

## 7. Establish GTM access and QA isolation

1. **Separate authentication from authorization.** A Google Ads login/token does not prove GTM API scopes or container permissions. Prefer an already authenticated, user-authorized browser for initial inspection; API use needs Google OAuth with the relevant Tag Manager scopes, not a PAT. Never invent an assistant email address to invite.
2. **Verify the account and visible containers.** With multiple Google sessions, inspect the active account and actual container IDs. `authuser` is a session selector, not a stable account identity. Seeing QA does not establish PROD access. Do not infer Edit/Publish permission from a visible Submit button alone.
3. **Request least privilege.** PROD Read is enough for comparison; QA Edit supports workspace changes but not creation of versions or publication. Approve permits version creation without publishing; Publish is needed to release a container version. Let the user publish if that is the agreed boundary. Export/preserve current state and use a dedicated workspace before authorized edits.
4. **Use the right browser session.** A headless automation tab is not necessarily visible on the user's Mac and does not inherit a native Chrome login. For a user-assisted native login and read-only AppleScript inspection, follow the Apple ecosystem skill's Chrome workflow; rediscover the target tab after the user logs in or closes tabs.
5. **Trace bootstrap dependencies before creating CMP tags or requesting Preview.** Inspect the application entry/root code and the shipped bundle for the conditions that load GTM and CookieScript. A missing GTM CMP tag does not mean no banner: the application may load it directly, or gate it to production. In a fresh controlled session, check loaded script URLs/resource entries, `window.CookieScript`, parsed consent categories without visitor identifiers, and Tag Assistant connection status. A configured container ID is not proof that `gtm.js` loaded. If GTM waits for prior consent while the banner exists only inside GTM, identify the circular dependency: the banner cannot run to obtain the consent required to load its container. A predicate requiring both analysis and marketing also blocks analysis-only visitors. Fix bootstrap ordering only alongside an audit of every tag's consent requirements and destinations; simply removing the loader gate can activate legacy tracking. Keep optional consent denied initially, one CMP installation path, QA-only code guards where required, and effective network/Consent-view tests before publication. A saved workspace or connected Preview alone is not an acceptance test.
6. **Check the CMP domain constraint.** Verify CookieScript's current subdomain policy before assuming another license is necessary. The vendor supports reuse of one banner and cookie report across a parent and its subdomains; a separate banner/report can count as another website. The `Remember consent for subdomains` setting is consent sharing, not merely adding an allowed hostname. Configure the parent without protocol or `www` only after confirming scope and approval.
7. **Keep shared-banner effects explicit.** Reusing a parent banner means banner configuration changes affect production, and consent choices can propagate between QA and PROD in the same browser. Use a dedicated browser profile, check existing host-only versus parent-domain cookies, and test first-visit and return-visit behavior. Reuse only if the banner/report accurately describes both environments; otherwise arrange a separate staging banner or subscription option.
8. **Keep measurement destinations separate.** QA must retain its own GTM container and PostHog project. Do not import production Ads conversions or analytics IDs merely to mirror consent behavior. Test app-side changes against the shared unchanged CMP where possible; treat any common CMP modification as a production-impacting change.

## 8. Report findings and request only necessary changes

Lead with measured activity and the People-versus-visitors explanation. State which identity behaviors were demonstrated and which remain hypotheses. List existing usable events, host contamination and consent observability gaps. Keep Wright's completion boundary at departure for Ovio; do not combine their projects.

For each proposed change, specify the responsible surface and minimum permission: source repository for SDK/events, consent manager for callbacks/categories, PostHog project settings for project configuration, and insight/dashboard write only to save reports. No write permission is needed for the read-only queries above.
