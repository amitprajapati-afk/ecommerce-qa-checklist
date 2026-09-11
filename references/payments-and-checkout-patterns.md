# Payments, Checkout & Buy Now Patterns for E-commerce QA

## Buy Now / Express Checkout Flow

"Buy Now" skips the cart page and takes the user straight to checkout with a single item (or
a small pre-set bundle). The most common production bug class here is that the offer engine,
which is usually wired into the cart page, gets bypassed entirely because Buy Now constructs
its own mini order object.

- Buy Now on a SKU that's part of an active BXGY/tier offer — does the offer still apply, or
  does Buy Now silently skip it (bug) vs. correctly show "add to cart instead to use this
  offer" (valid product decision, but must be explicit)?
- Buy Now quantity selector: does it respect the same min/max/stock constraints as regular
  add-to-cart?
- Buy Now while items already exist in the cart: does it check out only the Buy Now item, or
  does it merge with the existing cart silently (confusing if user didn't expect that)?
- Buy Now → back button → does the regular cart still reflect its pre-Buy-Now state
  correctly, uncorrupted by the Buy Now transaction?
- Buy Now on an out-of-stock item: CTA should be disabled/hidden, not throw an error at the
  checkout step.
- Coupon code entry on a Buy Now checkout: does the coupon module (usually built for the
  full-cart checkout) render and function correctly on this shorter flow?

## Payment Gateway States

Payment failures are rarely a clean "declined" — most real bugs live in the ambiguous states
between initiation and confirmation.

- **Authorized but not captured**: gateway confirms auth, but capture webhook is delayed or
  fails — does the order show as "processing" rather than falsely "confirmed" or falsely
  "failed"?
- **Idempotency on double-submit**: user double-clicks Pay, or hits back+forward during
  processing — verify only one order/charge is created, not two.
- **Webhook failure/delay**: simulate the payment gateway's success webhook arriving late or
  not at all — does the order stay stuck in a pending state indefinitely, or is there a
  reconciliation/polling fallback?
- **Timeout with unclear outcome**: network drops after payment submitted but before the
  response returns to the client — the user's real question is "was I charged?" — verify the
  order status page / support flow can answer this without relying on the user's memory of
  what they saw.
- **Price mismatch**: amount shown on the checkout summary vs. amount actually sent to the
  gateway — especially after a coupon/offer recalculation that happens client-side.
- **Refund on gateway timeout**: if a charge succeeds on the gateway side but the order
  creation fails on the storefront side, is there an automatic refund path, or does it require
  manual reconciliation? (Flag as a support/ops gap if untested.)

## BNPL / Pay Later (Simpl, LazyPay, ZestMoney, etc., if applicable)

- Eligibility check: does the BNPL option only show for eligible pincodes/order values/
  first-time vs. returning customers per the provider's rules?
- Approval flow: redirect to provider, approval, redirect back — verify order is created only
  after provider confirms, not optimistically before redirect-back completes.
- Provider rejection: user declined by BNPL provider — does checkout gracefully fall back to
  other payment methods, or dead-end?
- Partial BNPL + gift card/coupon: does the BNPL provider receive the correct post-discount
  amount to evaluate against its own limits?

## Partial / Conditional COD

- COD availability toggled by pincode — verify checkout hides/shows COD correctly per
  pincode, not just globally.
- COD order value cap: order above the cap should force online payment, with a clear message,
  not a silent COD option removal.
- Partial COD (pay a token amount online, rest on delivery) if supported: token amount
  calculation correctness, refund of token amount on COD order cancellation.

## Guest Checkout

- Full purchase completed without creating an account — verify order confirmation email/SMS
  still reaches the guest correctly (this is the #1 place guest flows break: notifications
  assume a logged-in user object).
- Post-order prompt to create an account: does it correctly pre-fill from the guest order
  data, and does the resulting account get linked to that guest order for order history?
- Guest checkout with an email that already has a registered account: merge into existing
  account, prompt to log in, or create a shadow duplicate account? (Duplicate accounts are a
  common bug and a support headache — flag if untested.)

## Cart Merge on Login

- Guest has items in cart → logs in mid-session, and the account also already has saved cart
  items — verify merge behavior is defined (union of both, prompt to choose, or one silently
  overwrites the other) and doesn't silently drop items.
- Merged cart re-validates stock/price/offer eligibility for all items, not just the newly
  logged-in account's original items.
- Wishlist merge follows the same pattern — see `discovery-engagement-patterns.md`.
