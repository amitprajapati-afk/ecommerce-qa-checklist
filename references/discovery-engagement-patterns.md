# Search, Wishlist & Gift Card/Loyalty Patterns for E-commerce QA

## Search & Autocomplete

Search bugs are disproportionately invisible in normal QA passes because a broken search
still "works" — it just returns the wrong or no results, which looks like a legitimate empty
state unless you know what should have matched.

- **Typo tolerance**: a query with a common misspelling (e.g. "shrit", "trouzers") should
  still surface the intended product — verify the tolerance threshold isn't so loose that it
  starts returning irrelevant matches for genuinely different words.
- **Synonym matching**: category-specific synonyms (e.g. "tee" → "t-shirt", "denims" →
  "jeans") — verify the synonym list is actually wired into the live index, not just
  documented and forgotten.
- **Zero-result query**: verify a real fallback (trending products, "did you mean", broader
  category suggestions) renders — a blank page or generic "no results" with no next action is
  a conversion-killing bug, not just a UX nitpick.
- **Search + filter combination**: apply a filter (e.g. price range, size) on top of a search
  query and verify the intersection is correct, not either filter or search winning silently.
- **Ranking/relevance**: an exact product-name match should outrank a partial/tangential
  match — test with a query that's an exact title of one product and a substring of several
  others.
- **Autocomplete/typeahead**: debounce timing (not firing a request per keystroke), recent
  searches shown for a logged-in user, trending/popular searches shown when the search box is
  empty/focused.
- **Out-of-stock items in search results**: verify they're still findable (with an OOS badge)
  rather than vanishing from the index entirely, which would make customers think the product
  was discontinued.

## Wishlist & Save for Later

- Add/remove from PDP and from listing/PLP — icon state (filled/outline) stays in sync across
  both surfaces for the same product.
- Persistence: logged-in wishlist survives across session, device, and app/web parity if both
  exist.
- **Guest wishlist merge on login**: same class of bug as cart merge — verify items added as
  a guest aren't silently dropped when the user logs in or creates an account.
- Move item from wishlist to cart: verify current price and stock are re-checked at the
  moment of the move, not carried over stale from when it was wishlisted.
- Price-drop or back-in-stock notification for a wishlisted item, if the platform supports it:
  triggers correctly, and doesn't fire repeatedly for the same price/stock state.
- Wishlist item that gets fully discontinued: shown with a clear "no longer available" state
  rather than a broken link or silent removal the user never notices.

## Gift Cards, Store Credit & Loyalty

- **Purchase a gift card as a product**: itself goes through cart/checkout like any SKU —
  verify it's excluded from physical-shipping flows (no shipping charge, no tracking number
  expected) and delivered correctly (email to self or a specified recipient).
- **Redeem gift card as a payment method**: partial balance redemption — remaining balance
  correctly carries forward to the next order; verify the balance can't go negative and that
  redeeming more than the order total doesn't create a "change due" bug.
- Gift card expiry: correctly blocks redemption past expiry with a clear message, and (per
  policy/regulation in the applicable market) expired balance handling is intentional, not a
  silent data loss.
- Gift card + coupon + loyalty points stacking: see the stacking matrix in
  `offer-logic-patterns.md` — the ordering of application (which discount calculates against
  which base amount) is the most common source of bugs across all three.
- **Loyalty points earn**: correct rate applied per order (may vary by product category,
  membership tier, or promotional multiplier periods) — verify points are awarded only after
  the order is confirmed/non-cancelled, not at the moment of placement.
- **Loyalty points redemption**: minimum redemption threshold enforced, redemption value per
  point matches the published rate, and redeemed points can't exceed the order's eligible
  amount (e.g. some programs exclude points redemption on already-discounted items).
- **Refund/cancellation impact on points**: if an order is returned or cancelled after points
  were earned, verify those points are correctly clawed back; if points were redeemed as part
  of that order's payment, verify they're correctly restored to the customer's balance rather
  than lost.
- Points/balance expiry countdown shown to the customer accurately reflects the actual
  backend expiry logic — a common gap is a UI that shows a static policy statement instead of
  the customer's actual computed expiry date.
