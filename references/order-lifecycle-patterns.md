# Order Management, Returns & Shipping Patterns for E-commerce QA

## Order Management & Tracking

- **State machine**: placed → confirmed → packed → shipped → out for delivery → delivered,
  with failed/cancelled as branches. Test every valid transition and, just as important, every
  invalid one (e.g. can a "delivered" order be moved back to "packed" via a bug in the
  webhook handler? It shouldn't be possible.)
- Order history list: pagination correctness at boundary (exactly page-size count, one over),
  filter by status/date range, correct thumbnail/item detail per line.
- Multi-item order where items ship separately (split fulfillment): each sub-shipment tracks
  and updates independently; overall order status reflects the least-advanced sub-shipment
  correctly (e.g. not "delivered" until all parts are delivered).
- Cancellation: allowed only within the valid window (e.g. before "packed"); cancelling after
  that point should show why it's blocked, not just disable the button with no explanation.
  Verify the linked refund is triggered automatically and correctly for the payment method
  used.
- Invoice: generation timing (immediately vs. on shipment), GST/tax breakup line-item
  correctness for Indian orders, HSN code presence if applicable, download and email delivery
  both produce identical figures.
- Address edit after order placed: allowed only pre-shipment; verify the shipping label
  actually regenerates with the new address rather than the edit only updating the database
  record cosmetically.
- Reorder / buy again from order history: correctly re-adds current-price, current-stock
  state of items — not the frozen historical price.

## Returns, Exchange & Refunds

- **RMA window**: return request allowed only within the policy window from delivery date —
  test at the boundary (last valid day, first invalid day) and note that "delivery date" vs.
  "order date" is a common off-by-window-length bug.
- Pickup-based return vs. self-ship: correct instructions/label generated for whichever mode
  applies to that pincode/product category.
- **Exchange for different variant** of the same SKU (e.g. size swap): verify stock is
  reserved for the incoming variant before the outgoing item is even received, or that the
  flow explicitly handles the incoming variant going out of stock mid-process.
- **Partial return** of a multi-item order: refund amount recalculates correctly for only the
  returned items, and any order-level discount (e.g. a coupon that applied to the whole cart)
  is prorated correctly rather than either double-refunded or lost entirely.
- Refund destination choice: original payment method vs. store credit — verify the user's
  choice is honored, and that COD orders (no original payment method to refund to) correctly
  default to store credit or bank transfer collection.
- Refund timeline states: initiated → processing → completed, each with a customer-visible
  timestamp; verify the "completed" state is only set once the payment gateway/bank actually
  confirms, not optimistically at initiation.
- Non-returnable item categories (e.g. innerwear, customized/engraved items): CTA correctly
  hidden or disabled for these, with policy reasoning shown, not just a generic error.
- Return abuse/fraud guardrails if applicable: repeated returns from same account flagged or
  rate-limited — note as a P2/P3 if the client has such a policy.

## Shipping & Delivery

- **Pincode serviceability**: checked at PDP (informational), cart, and checkout (blocking) —
  verify consistency across all three; a pincode marked serviceable on PDP but rejected at
  checkout is a common, confusing bug.
- COD availability varies by pincode independently of general serviceability — test a pincode
  that's serviceable for prepaid but not COD.
- Free shipping threshold: below, at, and above the threshold; verify the threshold is
  calculated on the correct base (pre-coupon vs. post-coupon — see stacking notes in
  `offer-logic-patterns.md`).
- Multiple shipments for a split/multi-warehouse order: each has its own tracking ID and
  carrier, correctly surfaced to the customer without confusion about "why are there two
  tracking links."
- Delivery ETA display: shown per-pincode and per-product (some SKUs may have longer
  fulfillment times) — verify it's not a single hardcoded value across the whole catalog.
- Serviceability change after order placed but before shipment (e.g. pincode gets
  deactivated by ops): verify there's a defined path (auto-cancel + refund, or manual ops
  flag) rather than the order silently stalling with no customer communication.
