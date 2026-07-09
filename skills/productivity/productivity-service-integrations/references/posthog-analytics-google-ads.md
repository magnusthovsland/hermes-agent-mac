# PostHog analytics and Google Ads destination checks

Use this reference when the user asks the agent to inspect PostHog analytics, event logs, or Data pipeline → Destinations such as Google Ads Conversions.

## Credential hand-off pattern

- If the user deliberately stores a PostHog key in Google Drive, retrieve it via Drive export/download without printing the secret.
- Validate the key with a narrow read-only call before saving it locally.
- Persist local credentials under `~/.hermes/credentials/` with `chmod 600`; never commit or echo full keys.
- Personal API keys usually start with `phx_`; project/capture keys usually start with `phc_`.
- PostHog private API host must match region, e.g. `https://eu.posthog.com` for EU.

Example local credential shape:

```json
{
  "host": "https://eu.posthog.com",
  "project_id": 29869,
  "project_name": "Teoria - Prod",
  "project_api_key": "phc_...",
  "personal_api_key": "phx_..."
}
```

## Useful API endpoints

- Current API key metadata/scopes: `GET /api/personal_api_keys/@current/`
- Project metadata: `GET /api/projects/:project_id/`
- HogQL query: `POST /api/projects/:project_id/query/`
- Hog functions / destinations: `GET /api/environments/:environment_id/hog_functions/`
- Destination detail: `GET /api/environments/:environment_id/hog_functions/:id/`
- Destination logs: `GET /api/environments/:environment_id/hog_functions/:id/logs/`
- Destination metrics: `GET /api/environments/:environment_id/hog_functions/:id/metrics/`

In PostHog Cloud, the `environment_id` commonly matches the project ID for these endpoints.

## Google Ads Conversions destination review

1. List Hog functions and find `type = destination`, `name = Google Ads Conversions`.
2. Read the destination detail and inspect:
   - `enabled`
   - `inputs.customerId.value`
   - each `mappings[].name`
   - `mappings[].filters.events`
   - `mappings[].inputs.conversionActionId.value`
   - `mappings[].inputs.gclid.value`
   - `conversionValue`, `currencyCode`, `orderId`
3. Check the destination code/behavior for skip conditions. PostHog's Google Ads template can skip with `Empty \`gclid\`. Skipping...` when the resolved input is empty.
4. Read logs/metrics to distinguish "event did not match mapping" from "matched but skipped" and from Google API errors.

## HogQL diagnostics for conversion eligibility

Prefer aggregated counts and redacted samples. Do not print raw click IDs, emails, IPs, or person identifiers.

Example: purchase eligibility for a destination that requires `User activated access`, `saleSource = teoria`, and GCLID:

```sql
SELECT
  count() AS total,
  countIf(not empty(properties.gclid)) AS event_gclid,
  countIf(not empty(person.properties.gclid)) AS person_gclid,
  countIf(not empty(person.properties.$initial_gclid)) AS person_initial_gclid,
  countIf(not empty(coalesce(properties.gclid, person.properties.gclid, person.properties.$initial_gclid))) AS any_gclid,
  sum(toFloat(properties.priceInNok)) AS value_nok
FROM events
WHERE event = 'User activated access'
  AND timestamp >= now() - INTERVAL 30 DAY
  AND properties.saleSource = 'teoria'
```

Break down missing click IDs by plan/class/value:

```sql
SELECT
  properties.priceInNok,
  properties.periodInMinutes,
  properties.klasse,
  count() AS total,
  countIf(not empty(properties.gclid)) AS gclid,
  count() - countIf(not empty(properties.gclid)) AS missing_gclid
FROM events
WHERE event = 'User activated access'
  AND timestamp >= now() - INTERVAL 30 DAY
  AND properties.saleSource = 'teoria'
GROUP BY properties.priceInNok, properties.periodInMinutes, properties.klasse
ORDER BY total DESC
```

Compare events where the same business event is emitted under two names:

```sql
SELECT event, properties.saleSource, count() total,
  countIf(not empty(properties.gclid)) event_gclid,
  sum(toFloat(properties.priceInNok)) value_nok
FROM events
WHERE event IN ('User purchased access', 'User activated access')
  AND timestamp >= now() - INTERVAL 30 DAY
GROUP BY event, properties.saleSource
ORDER BY event, properties.saleSource
```

## Deep-dive diagnostics for attribution loss

When conversions look low, trace click IDs through the funnel instead of checking only the final conversion event. Use aggregated counts and redact URL parameters.

Coverage by key funnel event:

```sql
SELECT event, count() AS total, count(DISTINCT distinct_id) AS users,
  countIf(not empty(properties.gclid)) AS gclid,
  countIf(not empty(properties.gbraid)) AS gbraid,
  countIf(not empty(properties.wbraid)) AS wbraid,
  countIf(not empty(properties.gad_source)) AS gad_source,
  countIf(not empty(properties.gad_campaignid)) AS gad_campaignid
FROM events
WHERE timestamp >= now() - INTERVAL 30 DAY
  AND event IN ('$pageview', '$identify', 'User clicked get free trial',
                'User clicked register button on underside',
                'User clicked buy practice time', 'User completed registration',
                'User activated free tier access', 'User purchased access',
                'User activated access')
GROUP BY event
ORDER BY total DESC
```

Find whether click IDs are in the URL but missing as properties:

```sql
SELECT
  if(properties.$current_url LIKE '%gclid=%', 'url_has_gclid',
    if(properties.$current_url LIKE '%gbraid=%', 'url_has_gbraid',
      if(properties.$current_url LIKE '%wbraid=%', 'url_has_wbraid', 'no_clickid_in_url'))) AS url_clickid,
  count() total,
  countIf(not empty(properties.gclid)) prop_gclid,
  countIf(not empty(properties.gbraid)) prop_gbraid,
  countIf(not empty(properties.wbraid)) prop_wbraid,
  count(DISTINCT distinct_id) users
FROM events
WHERE event = '$pageview' AND timestamp >= now() - INTERVAL 30 DAY
GROUP BY url_clickid
ORDER BY total DESC
```

Inspect whether backend purchase events carry any session-entry attribution:

```sql
SELECT event, properties.saleSource, count() total,
  countIf(not empty(properties.$session_entry_gclid)) AS session_gclid,
  countIf(not empty(properties.$session_entry_gbraid)) AS session_gbraid,
  countIf(not empty(properties.$session_entry_wbraid)) AS session_wbraid,
  countIf(not empty(properties.$session_entry_gad_source)) AS session_gad_source,
  countIf(not empty(properties.$current_url)) AS current_url,
  countIf(not empty(properties.$initial_current_url)) AS initial_current_url,
  countIf(not empty(properties.$referrer)) AS referrer
FROM events
WHERE timestamp >= now() - INTERVAL 30 DAY
  AND event IN ('User activated access', 'User purchased access')
GROUP BY event, properties.saleSource
ORDER BY event, properties.saleSource
```

Check consent/feature-flag signals carefully. Feature flag property names containing `/` need backticks in HogQL:

```sql
SELECT event,
  count() total,
  countIf(properties.`$feature/consent` = true) AS consent_true,
  countIf(properties.`$feature/consent` = false) AS consent_false,
  countIf(isNull(properties.`$feature/consent`)) AS consent_null,
  countIf(not empty(properties.gclid)) AS gclid,
  countIf(not empty(properties.gbraid)) AS gbraid,
  countIf(not empty(properties.wbraid)) AS wbraid
FROM events
WHERE timestamp >= now() - INTERVAL 30 DAY
  AND event IN ('$pageview', '$identify', 'User completed registration',
                'User clicked buy practice time', 'User activated free tier access',
                'User activated access', 'User purchased access')
GROUP BY event
ORDER BY total DESC
```

## Pitfalls and interpretation

- A conversion destination may say it reads `person.properties.gclid`, but backend/server events with `$process_person_profile = false` often cannot rely on person properties. Verify with HogQL rather than assuming person fallback works.
- If the destination requires GCLID and purchase events only carry GCLID on a small fraction of events, Google Ads may receive only that fraction even though PostHog purchase counts are healthy.
- Check `gbraid`/`wbraid` separately. They may exist earlier in the funnel while the destination only accepts `gclid`. In that case the issue is not "no Ads click IDs exist"; it is "the final conversion event/destination does not carry or use them."
- Backend purchase/activation events often lack `$current_url`, `$initial_current_url`, `$referrer`, `$session_entry_gclid`, `$session_entry_gbraid`, and `$session_entry_wbraid`. Treat that as a strong sign that attribution was not forwarded from frontend/session/order metadata.
- `properties.` names with punctuation need HogQL escaping, e.g. `properties.`$feature/consent``; an unescaped slash is parsed as division or a nested field and gives misleading query errors.
- `orderId` blank means weaker deduplication; surface it as a recommendation, not an automatic change.
- Avoid raw logs that expose identifiers. Summarize counts, mapping names, conversion action IDs, event names, and redacted samples.
