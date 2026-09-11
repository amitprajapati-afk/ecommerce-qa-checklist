# Offer Logic Patterns for E-commerce QA

## BXGY (Buy X Get Y)
- Trigger: add X units of qualifying SKU → Y units auto-added or discounted
- Test matrix:
  - qty = X-1 → no offer
  - qty = X → offer applies
  - qty = X+1 → offer still applies (or applies twice if stackable)
  - Remove item → offer removed
  - Change qty below X → offer removed
- Edge: Y item is out of stock, Y item is itself on another offer

## BXAY (Buy X Amount, get Y% or ₹Y off)
- Trigger: cart value reaches ₹X threshold
- Test matrix:
  - cart = ₹X-1 → no discount
  - cart = ₹X exactly → discount applies
  - cart = ₹X+1 → discount applies
  - Remove item pushing cart below threshold → discount removed
- Edge: cart value includes/excludes shipping, tax

## Tier Pricing
- Multiple tiers: qty 1-2 = ₹500, qty 3-5 = ₹450, qty 6+ = ₹400
- Test every tier boundary (at, just below, just above)
- Mixed SKUs: does tier count apply per-SKU or across cart?
- Tier downgrade: remove items → price recalculates upward

## UTM Priority vs Manual Promo
- UTM offer in URL + no manual code → UTM offer applies
- Manual promo code entered + UTM in URL → which wins? (test both outcomes)
- UTM removed (navigate away) → offer persists or drops?
- Expired UTM campaign → no offer, no error vs. silent fallback

## Auto-Apply Logic
- Correct UTM param present → offer auto-applied on cart load
- Wrong/missing param → no offer, no error shown
- Auto-apply + manual code entered → conflict handling
- Page refresh with param → offer re-applies or persists?

## Offer Stacking / Exclusion
- Two offers on same SKU → only higher value applies (or blocked)
- Site-wide sale + product-level offer → behavior defined?
- Freebie + percentage discount → can both apply?

## Freebie Logic
- Cart qualifies → freebie auto-added to cart
- Freebie should not be removable by user (or shows warning)
- Freebie qty = 1 always (cannot be increased)
- Cart drops below threshold → freebie auto-removed

## Coupon Stacking Matrix (Coupon + Offer + Gift Card + Loyalty)

This is where the highest density of real production bugs lives — most storefronts define
each discount type in isolation and never explicitly test the combinations. For every pair
below, the expected behavior must be an explicit product decision, not an accident of
implementation order. If the product team hasn't defined it, flag it as a spec gap rather
than assuming a default.

- **Coupon + auto-applied offer (e.g. BXGY)**: does the coupon apply on top of the offer
  price, or is it blocked because an offer is already active? Test both directions of
  application order (offer-first-then-coupon, coupon-first-then-offer-triggered).
- **Two coupon codes**: entering a second code while one is applied — replaces, blocks with
  error, or (rare, usually a bug) stacks silently?
- **Coupon + gift card as payment**: gift card is a payment method, not a discount — verify
  the coupon discount is calculated on the pre-gift-card total, and the gift card is applied
  to the discounted total, not the original.
- **Coupon + loyalty points redemption**: same ordering question — coupon discount first,
  then points redeemed against the reduced total (most common correct behavior), or against
  original total (usually a bug that overpays the customer)?
- **Coupon + free shipping threshold**: does the coupon discount get counted toward or
  excluded from the free-shipping qualifying amount? (Excluding it is usually correct but
  frequently implemented wrong.)
- **Site-wide sale + coupon**: many storefronts explicitly block coupon codes during sitewide
  sale periods — verify the blocking message is clear, not a silent no-op.
- **Removal cascade**: removing the item that triggered an auto-offer should also correctly
  re-validate any coupon that depended on the discounted total (e.g. a coupon with a minimum
  cart value — does it get auto-removed if the offer removal drops cart below the minimum?).
- **Expiry mid-session**: coupon expires while sitting in an open cart tab — does the next
  action (qty change, checkout) trigger revalidation and removal with a clear message?
