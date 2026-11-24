# Package campaign prices using existing APIs

## 1. Pin the reference and scope

Read current branch heads and the relevant reference implementation before editing. Check the target repository's own PR-base convention; do not automatically copy Wright's QA workflow to another repository. Keep reference/backend inspection read-only when only White Label changes are approved.

Trace an actual displayed package through card, detail, selected-date summary and login handoff. Record tenant ID, office ID, office-package ID and global product ID separately: they are different namespaces. Read environment-derived API bases and headers rather than guessing from a public domain.

## 2. Establish the money contract

The inspected Ovio package-list contract uses integer minor units:

| Field | Meaning |
| --- | --- |
| `packagePrice` | Ordinary price of this office's package |
| `discountedPrice` | API-calculated effective discounted amount; nullable and may equal the ordinary amount without an automatic campaign |
| `hasAutomaticDiscount` | Signal that the automatic-campaign presentation applies |
| `price` | Legacy effective price |
| `originalPrice` / `totalPriceOfContents` | Sum of individual contents, not the campaign before-price |

Reconfirm this mapping against the deployed version. A value in `discountedPrice` alone does not prove an automatic campaign. With the flag and both amounts present, render `Nå = discountedPrice`, `Førpris = packagePrice`; any campaign saving is their difference. Preserve existing ordinary-package savings thresholds outside campaign mode. Convert minor units only at the existing formatting boundary; do not recompute backend rounding, percentages or eligibility.

The existing formatter may return an empty string for zero. Handle zero-valued campaign amounts explicitly without changing global formatting behavior for unrelated screens. A zero saving need not add a noisy savings badge.

## 3. Resolve list/detail asymmetry without changing the API

The inspected Wright flow reads `/public/office-packages/all` filtered by `office_id` and `product_id` for package display. White Label uses the list for cards but `/public/office-packages/<office-package-id>` for richer detail. The latter can expose the campaign flag and effective legacy price without the separate normal/discounted fields.

Prefer this narrow enrichment boundary:

1. Fetch detail through the existing tenant-aware client.
2. If campaign fields are already sufficient, avoid another request. If automatic discount is explicitly false, retain the ordinary path.
3. When enrichment is needed, use the detail's `office.id` and `productId` to query the existing list, with the same tenant headers. Add accurate optional typing if older detail responses can omit those identifiers; do not guess a substitute.
4. Select the exact office-package `id` and global `productId`. Office scoping is supplied by the request; also check response office metadata if available.
5. Merge only the required pricing fields. Never replace rich detail content with a list item or alter selected-course/login parameters.
6. On unavailable, missing or mismatched results, preserve legacy detail pricing and suppress unsupported before-price claims. Do not hide failure by fabricating prices from content totals.

Tests must execute the actual loader with controlled network boundaries, not just test a detached price helper. Include wrong product/office, no matches, missing identifiers, request failure and selected-course invariance.

## 4. Verify three different claims separately

### Presentation

Use actual White Label card and summary components with their real styles. Cover campaign, ordinary package, old response, manual code not applied, partial fields and free discounted price. Check both responsive placements and the state after choosing a course date. Keep shared standalone-course presentation unchanged. Use a fixture where package-content value differs from normal package price so accidental `originalPrice` use fails visibly.

### Backend quotation

The inspected `/api/public/checkout` implementation calculated a quotation without creating an order. Before using it in another version, inspect its controller/service again; the path or HTTP method alone does not establish read-only semantics. Carry the same tenant, office-package ID, course choice and certificate used by the display.

Compare `toBePaidAmount` with the API effective price and rendered Now amount. A quote's aggregate `discount.amount` can include the contents-to-package reduction as well as the campaign, so it must not be used as campaign-only savings.

### Actual authenticated checkout

A public quote is not proof of an authenticated legacy cart. Student-specific discounts, credit and cart state may change the payable amount. Verify only with authorized test context and stop before order/reservation creation, payment-session creation or completion actions. If unavailable, report authenticated checkout as unverified rather than implying that a fixture or public quote satisfies it.

## 5. Deliver a reviewable frontend-only draft

Run the pinned package manager with its frozen lockfile, the project's lint/format gates, targeted tests, full build and typecheck. Compare existing type diagnostics against the unmodified baseline instead of fixing unrelated code. Include campaign/ordinary screenshots and label synthetic data. State exactly which quote and checkout boundaries were exercised. Draft creation is not permission to merge or deploy, and screenshots alone are not proof of live API correctness.
