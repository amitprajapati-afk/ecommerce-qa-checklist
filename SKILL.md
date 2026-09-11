---
name: ecommerce-qa-checklist
description: >
  Generates comprehensive, structured QA test checklists for e-commerce platforms covering
  13 modules across the full customer journey: Cart Logic (BXAY, BXGY, Tier Pricing), Add to
  Cart & Buy Now, Checkout & Payments (incl. BNPL, gateway states), Promo Codes & Coupon
  Stacking, Mystery Box / Freebies, Product Listing & PDP, Search & Autocomplete, Login & Auth
  (incl. Guest Checkout), Order Management & Tracking, Returns/Exchange/Refunds, Gift Cards/
  Store Credit/Loyalty, Shipping & Delivery, and Wishlist. Always use this skill when the user
  asks to generate test cases, checklists, or test suites for any e-commerce feature — even
  partial ones like "give me cart test cases", "test cases for buy now", "returns flow QA", or
  "what should I test for checkout". Output is always BOTH a Markdown table AND a Google Apps
  Script that populates a Google Sheet. Especially relevant for Shopify/Next.js storefronts,
  Indian D2C e-commerce platforms, and any product with offer/promo/discount/loyalty logic.
---

# E-commerce QA Checklist Generator

Generates structured test case checklists covering the full e-commerce customer journey —
not just cart and checkout, but discovery, payment, post-order, and retention flows too. A
narrow "just checkout" or "just cart" mental model misses where D2C bugs actually cluster:
coupon+offer+gift-card stacking conflicts, buy-now bypassing cart-level validation, refund
timelines, and search relevance.

**Always output TWO formats:**
1. **Markdown table** — for quick review and documentation
2. **Google Apps Script** — ready to paste and run to populate a Google Sheet

---

## Supported Modules

| Module | Prefix | Trigger Keywords |
|--------|--------|-------------------|
| Cart Logic | `CART` | cart, BXAY, BXGY, tier pricing, offer, discount, freebies, cart merge |
| Add to Cart & Buy Now | `ATC` | add to cart, buy now, quick add, express checkout, cart persistence |
| Checkout & Payments | `CHK` | checkout, payment, COD, UPI, EMI, BNPL, gateway, order placement |
| Promo Codes & Coupon Stacking | `PROMO` | promo code, coupon, UTM, campaign, auto-apply, stacking |
| Mystery Box | `MYS` | mystery box, blind box, collection config |
| Product Listing & PDP | `PDP` | PDP, product page, listing, variants, filters |
| Search & Autocomplete | `SRCH` | search, autocomplete, typeahead, zero results, typesense, algolia |
| Login & Auth (incl. Guest Checkout) | `AUTH` | login, signup, OTP, session, logout, guest checkout |
| Order Management & Tracking | `ORD` | order status, tracking, order history, cancellation, invoice |
| Returns, Exchange & Refunds | `RET` | return, exchange, refund, RMA, store credit refund |
| Gift Cards, Store Credit & Loyalty | `GC` | gift card, store credit, loyalty, points, redeem |
| Shipping & Delivery | `SHIP` | shipping, delivery, pincode, serviceability, ETA, free shipping |
| Wishlist & Save for Later | `WISH` | wishlist, save for later, favorites |

If the user doesn't specify modules, **generate for all 13** with a summary count per module.
If the request clearly targets one flow (e.g. "buy now test cases"), generate only that module
in depth rather than padding with unrelated ones — depth over breadth when scope is narrow.
For a full-suite request, it's fine to note the total case count is large (~250-350 across 13
modules) and offer to prioritize P1-only first if the user wants a faster pass.

---

## Test Case Schema

Each test case must have these fields:

| Field | Description |
|-------|-------------|
| `TC_ID` | Format: `PREFIX_XXX` e.g. `CART_001`, `ATC_003`, `RET_012` |
| `Module` | One of the 13 modules above |
| `Sub-Feature` | e.g. "BXGY Offer", "Buy Now CTA", "RMA Pickup" |
| `Test Case Title` | Clear action + expected outcome |
| `Pre-conditions` | What must be true before the test |
| `Test Steps` | Numbered, concise steps |
| `Expected Result` | Specific, assertable outcome |
| `Type` | Positive / Negative / Boundary / Edge |
| `Priority` | P1 / P2 / P3 |
| `Automation Status` | Automatable / Manual Only / Needs Data Setup |

---

## Module-Specific Rules

### Cart Logic (BXAY, BXGY, Tier Pricing)
- Always include: offer applied, offer NOT applied, offer removed on qty change
- Test offer stacking vs. exclusion; price recalculation on qty update and item removal
- Boundary: exactly at threshold qty, one below, one above
- Negative: expired offer, ineligible SKU, out-of-stock item with offer
- Guest cart → logged-in cart merge: conflict resolution when both carts have items
- Deep matrices: `references/offer-logic-patterns.md`

### Add to Cart & Buy Now
- Add to cart from: PDP, listing/PLP, search results, wishlist, recommendations widget
- Quick-add (no variant selected) vs. full add (variant required) — correct validation
- Buy Now: skips cart, goes straight to checkout — verify it still applies offers/exclusions
  correctly rather than bypassing offer engine entirely (common bug: Buy Now ignores BXGY)
- Add OOS item → blocked with clear messaging, not a silent failure
- Cart persistence: refresh, close tab, different device (logged-in), session expiry
- Cart badge/count updates instantly and correctly across add/remove/qty-change
- Deep matrix: `references/payments-and-checkout-patterns.md` (Buy Now flow section)

### Checkout & Payments
- Cover: COD, UPI, Credit/Debit Card, Wallet, EMI, BNPL (Simpl/LazyPay/ZestMoney if applicable)
- Payment failure → retry without duplicate order creation; partial payment; price mismatch
  between cart total shown and amount charged
- Gateway states: authorized-but-not-captured, webhook delay/failure, double-charge/idempotency
  on double-submit, timeout with unclear outcome ("did it go through?")
- Address: new address, saved address, invalid pincode, address edit after order placed
- Edge: session timeout mid-checkout, browser back button after successful payment
- Guest checkout: full flow without account, then optional post-order account creation
- Deep matrix: `references/payments-and-checkout-patterns.md`

### Promo Codes & Coupon Stacking
- Valid code → correct discount applied; invalid / expired / already-used → proper error
- UTM vs. manual promo: which takes priority when both present?
- Auto-apply: triggers on correct URL param, removed on param absent, persists on refresh
- **Stacking matrix is the highest-bug-density area** — test every combination: coupon +
  auto-offer, coupon + gift card, coupon + loyalty points, two coupons at once (should be
  blocked or explicitly allowed?)
- Deep matrix: `references/offer-logic-patterns.md` (Coupon Stacking Matrix section)

### Mystery Box
- Collection-level config: enabled vs. disabled state
- UTM-triggered reveal vs. default state; out-of-stock Mystery Box behavior
- Add to cart, qty change, remove; price display: hidden vs. revealed

### Product Listing & PDP
- Filters & sorting on listing page; variant selection (size, shade, pack) → price update
- Add to cart from listing vs. PDP; out-of-stock: disabled CTA, back-in-stock notify
- Images, descriptions, reviews widget load correctly; PDP deep-link opens correct variant

### Search & Autocomplete
- Typo tolerance (e.g. "shrit" → "shirt" results); synonym matching (e.g. "tee" → "t-shirt")
- Zero-result query: fallback suggestions shown, not a blank page
- Search + filter combination narrows correctly; ranking/relevance for exact vs. partial match
- Autocomplete: debounce behavior, recent searches, trending/popular searches shown when empty
- Deep matrix: `references/discovery-engagement-patterns.md`

### Login & Auth (incl. Guest Checkout)
- OTP login: valid, invalid, expired OTP, resend-OTP rate limiting
- Social login (if applicable); session persistence: refresh, new tab, browser close
- Logout: session cleared, redirect correct
- Guest checkout → post-order account creation prompt; guest wishlist/cart merge on login
- Deep matrix: `references/payments-and-checkout-patterns.md` (Guest Checkout section)

### Order Management & Tracking
- Order states: placed → confirmed → packed → shipped → out for delivery → delivered;
  also failed/cancelled paths
- Order history: pagination, filter by status/date, correct item-level detail
- Cancellation: allowed only within valid window, reflects correctly in payment refund flow
- Invoice: generation, GST/tax breakup correctness, download/email delivery
- Post-order address edit: allowed only pre-shipment, blocked after dispatch
- Deep matrix: `references/order-lifecycle-patterns.md`

### Returns, Exchange & Refunds
- RMA creation: within valid return window, blocked outside window
- Pickup-based vs. self-ship return; exchange for different variant of same SKU
- Partial return of a multi-item order — remaining items/refund calculated correctly
- Refund destination: original payment method vs. store credit — user choice honored
- Refund timeline states surfaced correctly (initiated → processing → completed)
- Deep matrix: `references/order-lifecycle-patterns.md`

### Gift Cards, Store Credit & Loyalty
- Purchase a gift card as a product; redeem gift card as a payment method
- Partial balance redemption — remaining balance carries to next order correctly
- Balance check, expiry handling, loyalty points earn rate and redemption threshold
- Refund/cancellation impact on already-earned or already-redeemed points
- Deep matrix: `references/discovery-engagement-patterns.md`

### Shipping & Delivery
- Pincode serviceability check at cart/checkout; COD availability varies by pincode
- Free shipping threshold: below/at/above; multiple shipments for a split/multi-warehouse order
- Delivery ETA display accuracy; serviceability change after order placed but before shipment
- Deep matrix: `references/order-lifecycle-patterns.md`

### Wishlist & Save for Later
- Add/remove from PDP and listing; persists across session and device when logged in
- Move item from wishlist to cart; guest wishlist merges into account wishlist on login
- Price-drop or back-in-stock notification for a wishlisted item (if supported)
- Deep matrix: `references/discovery-engagement-patterns.md`

---

## Markdown Output Format

```markdown
## Module: Cart Logic

| TC_ID | Sub-Feature | Test Case Title | Pre-conditions | Expected Result | Type | Priority |
|-------|-------------|-----------------|----------------|-----------------|------|----------|
| CART_001 | BXGY Offer | Verify BXGY discount applied when qty threshold met | BXGY offer active for SKU_X | Discount line shows correct amount, total updated | Positive | P1 |
...
```

---

## Google Apps Script Output Format

Generate a complete, runnable Apps Script. Follow these rules:

```javascript
function populateQASheet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();

  // Always delete existing sheet if present, then recreate
  let sheet = ss.getSheetByName('MODULE_TC');
  if (sheet) ss.deleteSheet(sheet);
  sheet = ss.insertSheet('MODULE_TC');

  // Headers — freeze row 1
  const headers = [
    'TC_ID', 'Module', 'Sub-Feature', 'Test Case Title',
    'Pre-conditions', 'Test Steps', 'Expected Result',
    'Type', 'Priority', 'Automation Status'
  ];
  sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
  sheet.setFrozenRows(1);
  // DO NOT use setFrozenColumns() unless explicitly needed

  // Style headers
  const headerRange = sheet.getRange(1, 1, 1, headers.length);
  headerRange.setBackground('#4A90D9');
  headerRange.setFontColor('#FFFFFF');
  headerRange.setFontWeight('bold');

  // Data rows — use 2D array
  const data = [
    ['CART_001', 'Cart', 'BXGY Offer', 'Verify BXGY discount applied...',
     'BXGY offer active', '1. Add SKU\n2. Set qty to threshold\n3. Check cart',
     'Discount applied correctly', 'Positive', 'P1', 'Automatable'],
    // ... more rows
  ];

  if (data.length > 0) {
    sheet.getRange(2, 1, data.length, headers.length).setValues(data);
  }

  // Auto-resize all columns
  sheet.autoResizeColumns(1, headers.length);

  SpreadsheetApp.getUi().alert(`✅ ${data.length} test cases added to MODULE_TC sheet.`);
}

// Add to custom menu
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('QA Tools')
    .addItem('Populate MODULE_TC', 'populateQASheet')
    .addToUi();
}
```

**Critical Apps Script Rules (never violate):**
- Always use `setValues()` with a 2D array — never loop `setValue()` per cell
- Always null-check sheet existence before operations
- Never call `setFrozenColumns(0)` — it throws a runtime error
- Use `\n` for multi-line steps within a cell string
- Each module gets its own sheet tab and its own function
- If generating multiple modules, create one `onOpen()` that adds all items to the menu

---

## Multi-Module Output Structure

When generating all 13 modules, open with a summary table like this before the detail:

```
Generated: 13 modules, ~280 test cases total

| Module | Sheet Tab | TC Count | Function |
|--------|-----------|----------|----------|
| Cart Logic | Cart_TC | 30 | populateCartTC() |
| Add to Cart & Buy Now | ATC_TC | 18 | populateATCTC() |
| Checkout & Payments | Checkout_TC | 28 | populateCheckoutTC() |
| Promo & Coupon Stacking | Promo_TC | 24 | populatePromoTC() |
| Mystery Box | MysteryBox_TC | 15 | populateMysteryBoxTC() |
| PDP & Listing | PDP_TC | 22 | populatePDPTC() |
| Search & Autocomplete | Search_TC | 16 | populateSearchTC() |
| Login & Auth | Auth_TC | 20 | populateAuthTC() |
| Order Management | Order_TC | 20 | populateOrderTC() |
| Returns & Refunds | Returns_TC | 22 | populateReturnsTC() |
| Gift Cards & Loyalty | GiftCard_TC | 18 | populateGiftCardTC() |
| Shipping & Delivery | Shipping_TC | 16 | populateShippingTC() |
| Wishlist | Wishlist_TC | 10 | populateWishlistTC() |
```

Then output: Markdown tables first, then the full combined Apps Script.

---

## Output Checklist (verify before returning)

- [ ] Every TC has a unique TC_ID with correct module prefix
- [ ] Mix of Positive, Negative, Boundary, Edge cases per module
- [ ] P1 cases cover the most critical user flows
- [ ] Apps Script uses `setValues()` with 2D array (not per-cell loop)
- [ ] No `setFrozenColumns(0)` call present
- [ ] `onOpen()` menu includes all module functions
- [ ] Pre-conditions are specific, not vague ("offer is active" not just "setup done")
- [ ] Expected results are assertable (specific values, not "works correctly")
- [ ] If coupon/offer/gift-card/loyalty modules are all in scope, at least one stacking-conflict
      test case exists that combines two of them

---

## Reference Files

- `references/offer-logic-patterns.md` — Deep patterns for BXGY, BXAY, Tier Pricing, UTM
  stacking, and the Coupon + Offer + Gift Card + Loyalty stacking matrix. Read when generating
  Cart or Promo test cases for complex offer configurations.
- `references/payments-and-checkout-patterns.md` — Payment gateway states, BNPL, Buy Now /
  express checkout flow, guest checkout, cart merge on login. Read when generating Checkout,
  Add to Cart & Buy Now, or Auth test cases.
- `references/order-lifecycle-patterns.md` — Order Management states, Returns/Exchange/Refund
  flows, Shipping & Delivery serviceability logic. Read when generating Order, Returns, or
  Shipping test cases.
- `references/discovery-engagement-patterns.md` — Search/autocomplete relevance patterns,
  Wishlist persistence, Gift Card/Loyalty redemption logic. Read when generating Search,
  Wishlist, or Gift Card test cases.
