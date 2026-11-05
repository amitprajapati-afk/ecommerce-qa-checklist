# Shopify Checkout Variants: Native vs. Third-Party

Read this when the Platform Profile says the store is on Shopify. Which checkout is live
decides which engine applies discounts, which payment methods exist, and how the order lands
in Shopify admin. Tag checkout-dependent cases with `[Native Checkout]` or with the provider
name, e.g. `[GoKwik]`, `[Shiprocket Checkout]`, `[Razorpay Magic]`, at the end of the
Sub-Feature.

---

## Native Shopify Checkout

Discounts are evaluated by Shopify's own engine (admin discounts, automatic discounts, and any
Shopify Functions), and customizations come from checkout extensibility (checkout UI
extensions, payment/delivery customizations).

### Discount Engine `[Native Checkout]`
- Combination flags: for each discount pair in scope, verify they stack only when *both* are
  configured to combine with the other's class (product / order / shipping). Test one case
  where the business expects stacking but a flag is off — the expected result is
  "does not stack", and the case documents the config to fix.
- Automatic discount + manual code: verify which wins, or whether both apply, matches the
  combination settings; verify the checkout shows the correct discount lines, not only a
  reduced total.
- Discount eligibility limits: minimum purchase, minimum quantity, customer segment,
  once-per-customer, total usage limit — test at, below, and above each limit.
- Discount applied in cart and then cart changes at checkout (quantity edit, item removed via
  back navigation): verify the discount re-evaluates and is removed if no longer eligible.

### Payments `[Native Checkout]`
- Payment providers configured (e.g. Razorpay, PayU, Cashfree, other gateway apps):
  verify each method's redirect/return, and order creation only after confirmed payment.
- Manual payment methods (COD): verify COD shows only where the rules allow it (e.g. by
  pincode, cart value, or via payment customization), and order is created with payment
  status "pending".
- Use test mode / test gateway on a dev or staging store for automation; keep real-gateway
  flows as manual smoke tests.

### Checkout Customizations `[Native Checkout]`
- Checkout UI extensions (gift message, delivery date, upsell block): verify they render,
  their values save to the order (attributes/notes/metafields), and checkout still completes
  if the extension errors.
- Delivery/payment customization Functions (hide COD over a value, rename shipping method):
  verify rule boundaries — at, below, above the threshold.

### Order Creation `[Native Checkout]`
- Thank-you / order status page: correct order number, line items, discount lines, payment
  method, and address.
- Admin verification: order shows discount codes and allocations per line, correct tax
  breakup, correct tags, and inventory decremented once.
- Abandoned checkout: verify recovery email/link restores the same lines and discount.

---

## Third-Party Checkout (GoKwik, Shiprocket Checkout, Razorpay Magic Checkout, others)

The provider replaces the checkout UI and usually creates the order in Shopify through the
Admin API after payment. The highest-risk area is the **sync contract**: what the provider
shows the shopper vs. what lands in Shopify admin.

### Entry & Fallback `[Third-Party Checkout]`
- Checkout button from cart page, cart drawer, and Buy Now: verify each one opens the
  provider checkout, not native checkout on some entry points (common inconsistency).
- Provider script fails to load or times out: verify the defined fallback (native checkout or
  a clear error) — never a dead button.
- Cart contents, quantities, line item properties, and cart attributes carry into the
  provider checkout exactly.

### Discounts in the Provider `[Third-Party Checkout]`
- Shopify discount codes: verify the provider validates them against Shopify (same
  eligibility, limits, expiry) and doesn't accept a code that native checkout would reject.
- Automatic discounts and Shopify Functions–based offers (BXGY, tier pricing): verify they
  are applied in the provider checkout with the same amounts as the cart showed — this is a
  frequent mismatch, so test each active offer type explicitly.
- Provider-only incentives (prepaid discount, COD fee, coupon widget offers): verify the
  amount, verify how they stack with Shopify discounts, and verify they appear as a
  discount or fee line on the Shopify order, not just a lower total.
- Gift card and store credit redemption: verify whether the provider supports Shopify gift
  cards; if not, verify the UI blocks it clearly instead of failing at payment.

### Login, Address & COD `[Third-Party Checkout]`
- Phone OTP login: valid, invalid, expired OTP, resend limit; verify the Shopify order is
  linked to the correct customer (existing customer matched, not duplicated).
- Address prefill from the provider's network: verify the shopper can edit it and the edited
  address, not the prefilled one, reaches the order.
- COD availability rules (pincode, cart value, risk-based COD disable): verify each rule's
  boundary and the messaging shown when COD is hidden.

### Order Sync `[Third-Party Checkout]`
- After successful payment: verify exactly one Shopify order is created, with correct line
  items, prices, discount lines, shipping, tax, payment gateway name, financial status, and
  any tags/notes the provider adds.
- Delayed or failed sync (simulate slow webhook or retry): verify no duplicate order on
  retry, and the shopper sees a pending state rather than a false failure.
- Payment success but order creation fails: verify the refund/reconciliation path defined by
  the provider and that the shopper gets a clear message.
- Inventory: verify stock is decremented once, and the last unit can't be sold twice when two
  shoppers check out simultaneously.
- Abandoned cart recovery is often handled by the provider: verify the recovery link restores
  the cart and discounts, and doesn't conflict with Shopify's own abandoned checkout emails.

### Automation Notes `[Third-Party Checkout]`
- Provider checkouts usually load in a popup or iframe on another domain; prefer verifying
  the end state through the Shopify Admin API (order created, correct totals) over driving
  the full provider UI.
- Ask the provider for a sandbox/test mode; mark cases that need it as `Needs Data Setup`.

---

## When Checkout Is Unknown

Generate native checkout cases, add the Entry & Fallback and Order Sync cases tagged
`[Third-Party Checkout]`, and note in the summary that the user should confirm the provider
so provider-specific discount and COD cases can be added.
