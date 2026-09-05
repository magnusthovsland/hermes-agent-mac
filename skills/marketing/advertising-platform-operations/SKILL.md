---
name: advertising-platform-operations
description: "Use for paid-ad campaign changes through platform APIs."
version: 1.0.0
metadata:
  hermes:
    tags: [advertising, campaigns, ppc, api, operations]
---

# Advertising Platform Operations

## Overview

Use this class-level workflow for operational changes to paid-ad accounts through an API: account inspection, campaign/ad-group/ad creation, creative upload, targeting, bidding, activation, and delivery verification. Prefer official API documentation and current schemas over remembered behavior.

## Operating principles

1. Read the live account and existing hierarchy before writing.
2. Treat account/profile metadata, budget, bidding, targeting, creative, and activation as separate scopes.
3. Use idempotency keys and persist returned IDs after each successful write.
4. Create new structures paused unless the user explicitly authorized launch and a real budget.
5. Verify every write by retrieving the saved object.
6. Distinguish configured/approved/eligible from actual measured delivery.
7. Never expose API keys in commands, logs, summaries, or generated artifacts.

## Generic hierarchy workflow

1. Confirm account identity, currency, timezone, verification, and primary domain.
2. Resolve current targetable locations through the platform’s lookup endpoint.
3. Validate copy limits and image dimensions before upload.
4. Create campaign → ad group → ad in dependency order.
5. Keep language or intent segments in distinct ad groups when the user requests separate context/audience signals.
6. Retrieve all levels, including serving/eligibility diagnostics when available.
7. Activate only after budget and launch approval.
8. Check insights to confirm impressions, clicks, and spend after launch.

## Shared-account metadata safety

A campaign request does not automatically authorize changing shared advertiser identity. If a destination domain conflicts with the account profile, explain the dependency and obtain explicit approval before changing account display name, URL, favicon, legal representation, payment settings, or other account-wide metadata. Prefer a brand-specific account when appropriate.

## Verification language

- **Paused:** cannot deliver.
- **Active/configured:** enabled, but may still be blocked.
- **Approved/eligible:** can enter delivery if all parent objects and account checks pass.
- **Serving:** platform reports no current delivery blocker; still confirm with insights.
- **Delivered:** impressions or spend are present in reporting.

## Platform references

- OpenAI Ads Advertiser API: `references/openai-ads-advertiser-api.md`

## Completion checklist

- [ ] Correct account and domain verified.
- [ ] Budget and bidding match the user’s intent.
- [ ] Targeting is explicit, not inherited accidentally.
- [ ] Copy and creatives match platform limits.
- [ ] Tracking template is stored in the intended field.
- [ ] Objects were retrieved after creation/update.
- [ ] Status, review, serving issues, and insights were checked separately.
- [ ] No unapproved activation or account-wide metadata change occurred.
