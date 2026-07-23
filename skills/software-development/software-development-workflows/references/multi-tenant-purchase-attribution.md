# Multi-tenant SaaS purchase attribution

Use this reference when a tenant-owned marketing site sends a customer into a separate SaaS application where registration and payment occur, and each tenant wants purchase reporting in its own GA4 property.

## First decide the reporting destination

Do not conflate these requirements:

1. Purchase visible in the tenant's GA4 property.
2. Purchase attributed to the original organic/paid/session source in GA4.
3. Purchase uploaded directly to the tenant's Google Ads account.
4. Generic `dataLayer` event for tenant-controlled GTM or other vendors.

Ask which is required before proposing OAuth, Google Ads conversion uploads, cross-domain setup, or Measurement Protocol secrets.

## Core facts

- `dataLayer.push(...)` sends nothing by itself; GTM, Google tag, or another adapter must consume it.
- A GA4 `purchase` event describes what was purchased. It does not by itself preserve whether the visit was organic, paid, referral, or direct.
- Attribution continuity requires the original GA4 identity/session context, normally `client_id` and `session_id`, to follow the journey.
- Sending `gclid` as an arbitrary GA4 purchase parameter is not a substitute for preserving GA4 session identity. Direct use of `gclid` belongs to Google Ads click-conversion upload, which is a different integration.
- Browser-side Google tag/`gtag` needs only the tenant's public `G-...` Measurement ID; it does not need a Measurement Protocol API secret.
- Server-side GA4 Measurement Protocol always requires the tenant's Measurement ID and API secret. The secret is required because the call is server-side, not because the purchase occurred on another domain.
- Loading a tenant's Google tag in the SaaS does not mean loading every tenant's tag. Resolve the current tenant first and load only that tenant's Measurement ID, preferably with `send_page_view: false` when only purchase reporting is intended.

## Recommended architecture when cross-domain is unwanted

Do not automatically tell every tenant to add the SaaS domain to GA4 cross-domain settings. That can be inappropriate for a multi-tenant platform where the SaaS is intentionally separate from tenant websites.

Use a tenant-scoped attribution handoff:

1. On the tenant site, after analytics consent, obtain the GA4 identity/session required for continuity.
2. Transfer it to the SaaS using a short-lived, signed, opaque handoff token rather than exposing mutable raw attribution fields in URLs.
3. Bind the handoff to `tenant_id`, registration session, user, and eventual order.
4. On backend-confirmed payment, emit an idempotent internal `PurchaseCompleted` event.
5. Send GA4 `purchase` using one tenant-specific adapter:
   - browser adapter: dynamically initialize only the current tenant's `G-...` ID, suppress automatic pageviews, reuse the handed-off identity/session, and emit `purchase`; no API secret;
   - server adapter: use Measurement Protocol with encrypted tenant Measurement ID/API secret and the handed-off `client_id`/`session_id`.
6. Use a stable `transaction_id` and unique `(tenant_id, event_type, transaction_id)` constraint.
7. Never send student PII.

Validate current Google documentation before relying on exact `gtag get/config` fields or Measurement Protocol session semantics; these details can evolve.

## Purchase payload

Use GA4 recommended ecommerce fields:

- `transaction_id`
- `value`
- `currency`
- `items[]` with `item_id`, `item_name`, `item_category`, `price`, and `quantity`
- optional `affiliation`/department, tax, and other non-PII business metadata

Trigger only after the backend records a successful/captured payment, not on button click or redirect. Define invoice, balance, package, and refund semantics explicitly.

## Multi-tenant safeguards

- Validate Measurement IDs (`G-...`), and never accept arbitrary JavaScript.
- Resolve tenant before initializing analytics.
- Ensure tenant A's purchase can never route to tenant B's property.
- Encrypt API secrets and keep them server-side.
- Respect tenant/user consent before analytics collection.
- Test refresh, duplicate webhooks, multiple tabs, failed payments, tenant switching, and attribution continuity.

## Communication pitfall

Keep the answer aligned with the user's chosen destination. If the user asks only for GA4 purchase measurement, do not expand into direct Google Ads account connections. State the minimum required solution first, then mention alternatives only if requested.