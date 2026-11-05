# Shopify App Integrations: Loyalty, Search, Reviews & Returns

Read this when the Platform Profile lists the apps that power loyalty, search, reviews, or
returns. On Shopify these features are rarely native, so the most valuable test cases check
the **contract between the app and Shopify**: data in sync, discounts created correctly, and
the store still working when the app misbehaves. Tag app-dependent cases with the app name
at the end of the Sub-Feature, e.g. `[App: Judge.me]`. If the app is unnamed, use
`[App: Loyalty]`, `[App: Search]`, and so on.

Map these cases into the existing modules: loyalty → `GC`, search → `SRCH`,
reviews → `PDP`, returns → `RET`.

---

## Common to Every App

- App disabled, uninstalled, or its script blocked: verify the storefront and checkout keep
  working and the widget area doesn't break layout.
- Theme vs. headless: on a theme, verify the app embed/block is enabled on the published
  theme; on headless, verify the feature is wired through the app's API/SDK.
- Mobile and desktop rendering of the widget; no layout shift that moves the Add to Cart
  button.
- Customer identity: verify the app recognizes the logged-in Shopify customer and doesn't
  mix data between guest and logged-in sessions.

---

## Loyalty & Rewards (e.g. Smile.io, Yotpo Loyalty, BON, Nector, Flits) → `GC`

- Points earn: verify points are credited for an order per the earn rule, only after the
  configured trigger (order paid / fulfilled), and not for cancelled or fully refunded orders.
- Points redemption: verify redeeming creates a Shopify discount code or applies a discount
  of the exact value, the code is single-use and tied to that customer, and unused codes
  expire as configured.
- Redemption threshold: below, at, and above the minimum points; redemption more than the
  cart value.
- Loyalty discount stacking: verify how the generated discount combines with other coupons
  and automatic offers — it follows Shopify's combination flags on native checkout, and the
  provider's rules on third-party checkout.
- Refund/cancellation: verify earned points are reversed, and redeemed points are restored
  (or not) per the stated policy.
- Non-order earn actions (signup, birthday, referral, review): verify each awards points once
  and can't be farmed by repeating the action.
- Tier upgrades/downgrades: verify tier change at the exact spend threshold and that tier
  perks (e.g. free shipping) apply in checkout.

---

## Search (e.g. Shopify Search & Discovery, Boost, Searchanise, Algolia, Typesense) → `SRCH`

- Index sync: create, edit (title, price, tags), unpublish, and delete a product in admin;
  verify search results reflect each change within the app's stated sync window.
- Out-of-stock handling: verify OOS products are hidden, pushed down, or shown per config.
- Variant-level search: searching a variant attribute (shade, size) returns the parent
  product and opens the matching variant.
- Synonyms, typo tolerance, and pinned/boosted results configured in the app are actually
  live on the storefront.
- Filters from the app (price, collection, metafield-based filters): verify counts match
  results and that filters combine correctly with the search query.
- Market/currency: prices in search results and autocomplete match the shopper's market.
- Search results → Add to Cart (if supported): same variant and offer rules as PDP.
- Analytics: zero-result queries are captured in the app dashboard (useful for synonym
  tuning).

---

## Reviews & Ratings (e.g. Judge.me, Yotpo, Okendo, Loox) → `PDP`

- Review submission: text-only, with photo/video, with rating only; required fields and
  length limits; profanity/spam handling.
- Verified buyer: review from a post-purchase email link is marked verified; a review from a
  non-buyer is unverified or blocked per config.
- Moderation: a review held for approval doesn't appear until approved; rejected reviews
  never appear.
- Display: star rating average and count on PDP, PLP cards, and search results match the
  app dashboard; sorting and filtering reviews work.
- Variant/product grouping: reviews shared across grouped products or variants display as
  configured.
- Review request emails: triggered after the configured event (fulfillment/delivery), not for
  cancelled orders.
- Review incentives (points or a discount for reviewing): verify reward is issued once and
  links correctly with the loyalty app if both are used.
- SEO: product structured data includes rating values that match what's displayed.

---

## Returns & Exchanges (e.g. Shopify native returns, Return Prime, EcoReturns, ReturnGo) → `RET`

- Eligibility: return window from the correct start date (delivery vs. fulfillment),
  non-returnable products/collections blocked, final-sale and discounted items per policy.
- Order lookup: by order number + email/phone; verify a shopper can't open another
  customer's order.
- Shopify sync: return request creates the matching return/refund record in Shopify admin,
  with correct line items and quantities.
- Exchange: exchange for another variant creates the replacement order (or draft) with correct
  pricing; price difference is charged or refunded correctly; inventory reserved for the new
  variant.
- Refund destination: original payment method vs. store credit vs. gift card — verify the
  amount, and that a store credit or gift card issued by the app is usable at checkout.
- Discounted orders: refund amount accounts for the discount allocated to that line (partial
  return of a BXGY or coupon order must not refund more than was paid).
- COD orders: refund to bank/UPI details captured by the app; validation of those details.
- Pickup integration: reverse pickup booked with the logistics partner, status updates flow
  back to the app and the shopper.
- Loyalty impact: points earned on returned items are reversed.

---

## When Apps Are Unknown

Generate the "Common to Every App" cases plus generic loyalty, search, reviews, and returns
cases tagged `[App: Loyalty]`, `[App: Search]`, `[App: Reviews]`, `[App: Returns]`, and note
in the summary that naming the apps would allow app-specific cases.
