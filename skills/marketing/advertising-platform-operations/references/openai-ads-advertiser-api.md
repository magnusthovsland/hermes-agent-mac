# OpenAI Ads Advertiser API: operational notes

Use this when creating or verifying OpenAI Ads campaigns through the Advertiser API.

## Primary sources

- Overview: https://developers.openai.com/ads/api-overview
- Quickstart: https://developers.openai.com/ads/api-quickstart
- OpenAPI spec: https://developers.openai.com/ads/openapi.json
- Targeting: https://developers.openai.com/ads/campaign-targeting
- Base URL: `https://api.ads.openai.com/v1`

Prefer the live OpenAPI spec over prose when they differ; it may expose newer fields such as bidding strategy, daily budget, and landing-page configuration.

## Safe creation sequence

1. `GET /ad_account`; record account `name`, `url`, currency, timezone, `review`, and `account_integrity_review`.
2. Resolve locations with `GET /geo_lookup/search`; do not assume a stored location ID remains targetable.
3. Create the campaign **paused**, with an idempotency key.
4. Upload the local creative through multipart `POST /upload`; store the returned `file_id`.
5. Create each ad group **paused**, with its own context hints and idempotency key.
6. Create each ad **paused**, then retrieve it and verify title/body/URL/file/review status.
7. Retrieve campaign, groups, and ads with `include[]=serving_issues` before activation.
8. Activate only after the user explicitly confirms the real budget and launch.

Persist IDs after each successful step so partial failures can resume without duplicates. Also list by stable object name before creating on retry.

## Click campaigns and “Maximize results”

For a click objective, use `bidding_type: "clicks"` and a daily campaign budget. For the ad group:

```json
{
  "bidding_config": {
    "billing_event_type": "click",
    "strategy": "maximize_clicks"
  }
}
```

Important:

- `maximize_clicks` requires a **daily** campaign budget.
- Daily and lifetime budget fields are mutually exclusive; send only one.
- Runtime account/currency minimums can exceed the generic OpenAPI minimum. Report the server-enforced minimum and keep the campaign paused until the user confirms it.
- A fixed `max_bid_micros` defaults the group to `fixed_bid`; do not use that when the requested strategy is Maximize results/clicks.

## URLs and query parameters

Keep the creative URL clean and put tracking in `landing_page_configuration.query_string_template` at campaign, ad-group, or ad scope:

```json
{
  "landing_page_configuration": {
    "query_string_template": "utm_source=chatgpt&utm_medium=cpc&utm_campaign=example&utm_content={ad_id}"
  }
}
```

Do not concatenate the template onto `creative.target_url` when the user supplies it separately as “Query parameters.”

A destination on a different domain from `ad_account.url` can return `403: Account verification must be approved before using a different ad URL.` The API supports changing account URL via `POST /ad_account/brand`, but this changes shared metadata and starts/restarts brand review. Obtain explicit approval before changing account name, URL, or favicon; alternatively use the correct brand-specific ad account.

The visible advertiser header comes from `ad_account.name`, not the campaign name. “On behalf of” or legal-verification details may not replace that display name. If brand visibility matters, verify account name, URL, and favicon in a preview.

## Copy, images, and context hints

- `creative.title`: 3–50 characters.
- `creative.body`: maximum 100 characters; count exact Unicode, including newlines.
- A requested line break can replace a space with `\n` to preserve length.
- Square 1080×1080 PNG creatives are a known-good format; verify dimensions before upload and retrieve the saved `image_crop` afterward.
- Keep language-specific audiences in separate ad groups with only same-language hints.
- Re-read context hints after Ads Manager edits. Commas in a hint may be split into separate hint entries by UI processing; verify the resulting array.

## Is it actually live?

Check all layers:

- account `status` and both review fields;
- campaign and ad-group `status` plus `serving_issues`;
- ad `status`, `review_status`, and `serving_issues`;
- insights for impressions, clicks, and spend.

Use array syntax: `?include[]=serving_issues`.

Interpretation:

- Active + approved + no serving issues = eligible to serve and able to spend.
- Empty insights = no measured delivery yet, not proof that configuration is paused.
- Brand review and account-integrity review are distinct; report each separately.
