# Google Ads vs Ahrefs paid-traffic reconciliation

Use this when a user asks whether Ahrefs `Paid traffic`/`Paid search` movement reflects real Google Ads spend, budget, clicks, or campaign status.

## Core lesson
Ahrefs `Paid traffic` is an external visibility/traffic estimate, not the Google Ads ledger. A large Ahrefs drop can occur while Google Ads spend and clicks remain broadly stable. Treat Ahrefs as a directional third-party signal and reconcile against primary Google Ads data before concluding that ads were reduced.

## Recommended workflow
1. Parse the Ahrefs export and identify:
   - date range
   - `Paid traffic`
   - `Organic traffic`
   - `Impressions`
   - organic pages / ranking buckets where present
   - step changes in paid traffic
2. Pull Google Ads primary metrics for the same date range:
   - `segments.date`
   - `campaign.name`, `campaign.status`, `campaign.advertising_channel_type`
   - `metrics.impressions`, `metrics.clicks`, `metrics.cost_micros`, `metrics.conversions`
3. Aggregate by comparable periods, not just endpoints. Useful cuts:
   - pre-change baseline
   - transition days
   - high plateau
   - post-drop/reduced period
   - last 7 days
4. Compare percent changes in:
   - Ahrefs paid traffic average
   - Google Ads cost/day
   - Google Ads clicks/day
   - Google Ads impressions/day
   - conversions/day if enough volume
5. Pull current campaign budget/status to answer “have we actually turned this down?”:
   - `campaign_budget.amount_micros`
   - `campaign.status`
   - last-7-days cost/clicks by campaign
6. If useful, query `change_event` for campaign/budget edits, but remember Google Ads API change-event history is limited (commonly last 30 days). Do not overclaim if the suspected change is older.

## Interpretation rules
- If Ahrefs drops far more than Google Ads cost/clicks, say clearly that the Ahrefs movement does **not** match actual Google Ads reduction.
- If Google Ads spend is stable but Ahrefs paid traffic drops, likely explanations include Ahrefs keyword/sample coverage, observed SERP/ad visibility changes, geo/device/timing differences, competitor SERP changes, or keyword mix shifts.
- If Google Ads spend/clicks also drop proportionally, then the Ahrefs result may be consistent with a real budget/campaign reduction.
- Keep organic findings separate. Ahrefs organic changes may be relevant context, but they do not prove paid-search spend changed.

## Reporting pattern
Use a compact table:

| Period | Ahrefs paid avg | Google Ads cost/day | Google Ads clicks/day | Google Ads impressions/day |
|---|---:|---:|---:|---:|

Then state the mismatch in one sentence, e.g.:

> Ahrefs paid traffic fell ~94%, but Google Ads cost/day fell only ~6% and clicks/day only ~3%, so this is not primarily explained by ads being turned down.

## Google Ads API query skeleton

```sql
SELECT
  segments.date,
  campaign.id,
  campaign.name,
  campaign.status,
  campaign.advertising_channel_type,
  metrics.impressions,
  metrics.clicks,
  metrics.cost_micros,
  metrics.conversions,
  metrics.ctr,
  metrics.average_cpc
FROM campaign
WHERE segments.date BETWEEN 'YYYY-MM-DD' AND 'YYYY-MM-DD'
ORDER BY segments.date, campaign.name
```

For current budget/status:

```sql
SELECT
  campaign.name,
  campaign.status,
  campaign.advertising_channel_type,
  campaign_budget.amount_micros,
  campaign_budget.status,
  metrics.impressions,
  metrics.clicks,
  metrics.cost_micros,
  metrics.conversions
FROM campaign
WHERE campaign.advertising_channel_type = 'SEARCH'
  AND segments.date DURING LAST_7_DAYS
ORDER BY campaign.name
```

For account context:

```sql
SELECT customer.currency_code, customer.time_zone, customer.descriptive_name
FROM customer
LIMIT 1
```
