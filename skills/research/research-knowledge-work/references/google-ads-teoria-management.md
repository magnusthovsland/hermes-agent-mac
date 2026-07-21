# Teoria Google Ads management notes

Use when managing/auditing Teoria Google Ads, especially during a strict performance reset.

## Account/context

- Google Ads customer ID: `2642145925` (`Teoria Ads`).
- Credentials have historically lived under `~/clawd/credentials/` via symlink to shared credentials.
- Focus product/class for July 2026 reset: **Klasse B / personbil** before expanding to MC, mopedbil, moped.
- User correction: Teoria ad copy should use **“40 % stryker”**, not the 44 % figure from the communications platform.
- Weekday context: Sunday, Monday, Tuesday are often stronger; Friday and Saturday often weaker.

## Strict reset workflow

1. Pull primary Google Ads data before acting:
   - campaign/ad group/keyword/search term spend, clicks, impressions
   - conversion-action split: `Purchase`, `Free access activated`, `Sign up`
   - current conversion goals and campaign goals
2. Diagnose purchases separately from free activations. Treat free activations as a lead-quality signal, not proof of success.
3. Ensure bidding/goal optimization prioritizes purchases:
   - `customer_conversion_goal`: `PURCHASE~WEBSITE` should be biddable.
   - `SUBSCRIBE_PAID~WEBSITE` / free access and `SIGNUP~WEBSITE` should not be biddable unless there is an explicit lead-gen strategy.
   - `conversion_action.include_in_conversions_metric` may be immutable for imported click conversions; use customer/campaign conversion goals instead.
4. If performance is poor, cut waste before adding new creative:
   - segment spend, clicks, purchases, and secondary conversions by `segments.ad_network_type` before evaluating keywords; Search Partners can dominate cheap traffic while producing no purchases
   - disable Search Partners when they consume material budget without purchase conversions, while preserving Google Search delivery
   - pause broad/free campaigns that produce free users but no tracked purchase
   - pause weak ad groups and phrase keywords with high spend/no purchases
   - keep only high-intent Klasse B terms while rebuilding
   - compare enabled RSAs by purchase results and retain at least one eligible/approved ad per active ad group
5. After mutations, read back campaign network settings, budget, keyword/ad statuses, and policy approval. Newly created keywords may remain `UNDER_REVIEW`; report that separately from enabled/eligible items.
6. Use “Prøv gratis i 30 min” as CTA, not the main positioning. Core positioning should be understanding/value, e.g. “Først forstått. Så bestått.”

## GAQL snippets

### Conversion goals

```sql
SELECT customer_conversion_goal.category,
       customer_conversion_goal.origin,
       customer_conversion_goal.biddable,
       customer_conversion_goal.resource_name
FROM customer_conversion_goal
```

```sql
SELECT campaign_conversion_goal.campaign,
       campaign_conversion_goal.category,
       campaign_conversion_goal.origin,
       campaign_conversion_goal.biddable,
       campaign_conversion_goal.resource_name
FROM campaign_conversion_goal
```

### Conversion action split

Do not combine `segments.conversion_action_name` with unsupported metrics like clicks/cost in the same query. Query conversions by action separately:

```sql
SELECT campaign.name,
       ad_group.name,
       segments.conversion_action_name,
       metrics.conversions,
       metrics.all_conversions,
       metrics.conversions_value
FROM ad_group
WHERE segments.date BETWEEN 'YYYY-MM-DD' AND 'YYYY-MM-DD'
ORDER BY metrics.conversions DESC
```

For spend/clicks, use a separate query without `segments.conversion_action_name`.

## Creative/policy notes

- Prefer exact/phrase high-intent terms for Klasse B during reset.
- Negative intent to watch: `gratis` in paid purchase campaigns, competitor/free-test terms, `quiz`, `fasit`, `spørsmål og svar`, `norge4`, low-value “free only” searches.
- New platform RSA should emphasize:
  - “Først forstått. Så bestått.”
  - “Ikke pugging. Forståelse”
  - “Laget av trafikklærere”
  - “2500+ spørsmål”
  - “40 % stryker teoriprøven”
  - “Prøv gratis i 30 min”
- RSA headlines/descriptions are immutable through normal update in Google Ads API; to fix copy like `44 %` → `40 %`, create a corrected RSA and pause the old one.

## Pre-launch maintenance spend and pause decisions

When deciding whether to keep spending before a later professional marketing launch, do not justify budget with traffic alone. Pull current month, last 7/14 days, and campaign/keyword lifetime data, then calculate:

- spend, clicks, CPC;
- **Purchase** count, purchase CVR, purchase CPA, and trustworthy purchase value/ROAS;
- free activations and signups separately;
- projected monthly spend at the current daily rate;
- spend/no-purchase keywords and ad groups;
- whether anyone will actively use the data during the holding period.

Decision rules:

1. If purchase is the only biddable goal but conversion volume is near zero, more non-converting clicks do not create useful Smart Bidding learning. Do not recommend NOK 8k–12k/month merely to “keep Google warm.”
2. A pause does not delete campaign history, penalize SEO, or create an account punishment. The real costs are lost traffic/fresh search-term data and possible recalibration when campaigns restart. State these narrowly; do not overstate a mythical reset.
3. If professionals will restructure campaigns later, a new learning period is likely anyway. Poor current traffic may be less valuable than preserved budget.
4. Offer two explicit strategies:
   - **Pause fully** when tracking/funnel is uncertain or nobody can monitor and act.
   - **Maintenance budget** around NOK 1.5k–3k/month, limited to high-intent Klasse B exact/phrase terms, when fresh Landing Page Experience/search-term evidence is genuinely useful.
5. Before recommending maintenance spend, identify obvious waste such as high-spend terms with zero purchases. Free activations are supporting funnel signals, not revenue justification.
6. Keep recommendations read-only unless the user explicitly approves budget, pause, keyword, or campaign changes.

Report actual primary-account evidence rather than generic PPC advice. A useful summary shape is: `MTD spend → clicks → purchases → purchase CPA → reported value/ROAS → last-7-day purchases → projected monthly spend`.

## Daily monitoring report shape

Report in Norwegian:

1. Status since yesterday / last 1, 3, 7 days and MTD.
2. Purchases, CPA purchase, spend, revenue/value if trustworthy.
3. Free access vs purchase quality — do not call free activations success by themselves.
4. Search terms to negative, keywords/ad groups to pause, budget changes.
5. Weekday context: compare current day against expected weak/strong weekday pattern.
6. Specific actions taken or recommended today.
