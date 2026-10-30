# Attribution contract, retention and verification

## 1. Establish the contract before implementation

Record independently: allowed fields and sources, event consumers, consent gates, replacement semantics, lifetime, and deployment authorization. Preserve agreed choices; do not ask again merely because implementation revealed SDK gaps. Do not invent unresolved policy to get a green test.

For Wright, the agreed consent split is UTM context with analytics consent; advertising click IDs and advertising-cookie values with BOTH analytics and marketing consent. This is the user's chosen policy, not a legal conclusion or evidence that Teoria already enforces it on every path.

The agreed replacement model is:
- A new campaign replaces the current UTM set atomically; remove omitted old UTM fields so a new campaign cannot inherit another campaign's content or term.
- Untagged navigation preserves the current set; ordinary navigation is not a new campaign observation.
- Keep latest click references per platform separately. Replace the same platform's previous reference without clearing unrelated platforms.
- Treat Google's alternative identifiers as a related group rather than accidentally retaining an old gclid beside an unrelated new braid.
- Preserve observation/source metadata so an earlier platform reference is not mistaken for the source of the current campaign.
- Reread available source cookies; do not let an old SDK copy override a changed cookie or resurrect one that disappeared.
- Apply changes to future events; do not imply that already-ingested historical events are rewritten.

Explain these rules using two successive campaign examples before asking for approval. Keep policy questions distinct from implementation choices.

## 2. Inventory field names and namespaces

Test all six UTM fields: utm_source, utm_medium, utm_campaign, utm_id, utm_content, utm_term. Preserve opaque campaign IDs as strings, including leading zeros. Use documented custom_campaign_params for missing campaign fields instead of adding per-button captures; prove the installed SDK's behavior.

User-requested advertising inventory:
- Google/YouTube URL: gclid, gbraid, wbraid.
- Meta URL: fbclid; source cookies _fbc and _fbp; canonical event/API values fbc and fbp.
- TikTok URL: ttclid; source cookie _ttp; canonical value ttp.
- Snap URL: exact-case ScCid, mapped to sc_click_id; source cookie _scid mapped to sc_cookie1.
- Microsoft URL: msclkid.
- OpenAI URL: oppref.
- LinkedIn URL/cookie: li_fat_id. Downstream LinkedIn API identifier typing is a separate mapping, not a reason to rename raw browser evidence.

Keep source cookie names, internal event names and conversion API field paths separate. Raw fbclid is not a finished fbc value; a browser reference is not a click reference. Validate case-sensitive URL inputs, not a similar lowercase SDK key. Obtain scope approval before adding extra vendor cookies/pixels just because documentation lists them.

## 3. Audit reference implementations without inheriting assumptions

Pin the reference repository and lockfile. Inspect initializer, capture wrapper, cookie reader, server synchronization and storage implementation separately. Verify how callers use helpers, not just whether a helper contains a consent check. When asked to match Teoria, present its actual implemented subset and reuse opportunities, then list separately any stricter Wright requirements it does not establish. Source inspection is neither an end-to-end pass nor proof of the same live defect; run the same isolated scenario against the reference configuration before making behavioral equivalence claims.

Do not derive a universal retention policy from Teoria's custom gclid cookie. A synchronization fingerprint cookie contains a hash for deduplication; its expiry does not expire raw PostHog event/person properties. Initial/latest person properties are not automatically the context attached to every subsequent event.

Separate four lifetimes:
1. The actual source cookie's expiry and refresh behavior.
2. The advertising platform's attribution/matching window.
3. Wright's own browser copies.
4. Historical events/person properties retained by PostHog.

Inspect actual persistence serialization and read-time expiry. A cookie_expiration setting or register(props, days) does not by itself prove localStorage values expire. Verify both stores; use controlled elapsed-time tests after a lifetime is approved. Navigation must not silently renew a fixed-from-observation policy.

Use Wright's approved 90-day rule for its own campaign and per-platform click context, measured from the relevant campaign/click observation and not extended by ordinary navigation or direct return. Reread source advertising cookies without additional durable copies that outlive them. Treat this as the approved product contract, not evidence it is already implemented or a guarantee of legal compliance.

For other retention decisions, base the proposal on the click-to-purchase journey, not campaign duration alone or an arbitrary default. Separate browser retention from platform conversion windows and PostHog historical retention; retaining a reference does not guarantee an ad platform will credit the conversion. Verify current authoritative legal guidance before compliance claims, explain necessity and transparency rather than declaring a fixed number universally compliant, and do not imply every field requires a custom technical expiry engine.

## 4. Build independent acceptance scenarios

Use the locked distributed SDK and real initializer/capture helper, with local transport interception before live QA. Establish RED before a narrow fix and retain the full contract's failing exit status.

Cross the available fields with pageview, native autocapture and named actions. Shared-helper tests prove the enrichment path, not every rendered handler; retain separate real UI evidence.

Scenarios:
- Tagged landing, untagged SPA navigation, same-tab reload, no-opener new tab, browser-context revisit.
- New partial UTM set after a complete campaign; assert no stale UTM leftovers.
- New same-platform click with a positive old-click control; assert unrelated platform references survive.
- Cookie-only input, changed cookie, deleted cookie, URL/cookie conflict with explicit source precedence.
- No choice, rejection, targeting-only, analytics-only, both categories, loss of targeting while analytics remains, full withdrawal and regrant.
- Failed/in-flight event followed by targeting withdrawal; inspect whether the old marketing payload retries.
- Referrer-only marketing values; inspect raw URL/referrer, session properties, event fields, person envelopes and browser storage.
- Elapsed expiry separately from close/reopen; reopening immediately is not a retention-duration test.

For fixed-lifetime context, remember which URL signatures the current document has already handled as well as the shared persisted latest signature. Otherwise an unchanged older tagged tab can overwrite a newer tab's campaign on every polling tick. Verify this with two independent adapters and real cross-tab consent transitions.

When simulating exact expiry with Python Playwright, numeric clock timestamps are seconds while JavaScript Date.now() uses milliseconds; explicitly convert or use datetime, and freeze the clock before asserting exact observation timestamps.

When the user explicitly parks known SDK defects, retain raw failing assertions and emit a separate exact (case, field) release-gate classification. Do not waive a whole storage or withdrawal category; assert the other storage slots and every newly captured payload remain clean.

Require a successful positive cookie capture before accepting its negative withdrawal case. Missing initial capture is a failed precondition, not proof that withdrawal works. Check configured custom fields independently on events and initial-person/session fields; those code paths may have different parameter lists.

Do not conflate a synthetically seeded cookie with an actual pixel's serialization or CMP inventory. Label local fixtures, live UI actions, server ingestion and downstream conversion proof separately.

## 5. Prepare CookieScript disclosures after actual storage is known

Provide Norwegian purpose drafts for confirmed source cookies and any newly introduced first-party storage. Describe the identifier's purpose, Wright's additional analytics use and the approved consent gate. Report localStorage as localStorage, not an invented cookie.

Populate provider, domain, actual name and expiry from the real setter/inventory; do not infer them from URL parameter names or advertising attribution windows. Do not publish entries for cookies not actually set. Reading an existing cookie does not itself establish that a new pixel was installed. Keep changes to shared CookieScript configuration separately authorized.

## 6. Deliver accurately

A tested utm_id extension is a completed narrow feature, not a completed attribution system. Report the implemented slice, concrete remaining failure classes and the one decision that blocks further implementation. Do not hide unrelated red checks with a green narrow test command. Keep Wright attribution, Ovio handoff, actual receiver persistence and production activation as separate deliverables.

Primary starting references, to reread before relying on current behavior:
- https://posthog.com/docs/data/utm-segmentation
- https://posthog.com/docs/libraries/js/persistence
- https://posthog.com/docs/references/posthog-js/types/PostHogConfig
- https://developers.openai.com/ads/measurement-pixel
