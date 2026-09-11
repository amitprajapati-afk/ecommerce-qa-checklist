# ecommerce-qa-checklist

A Claude Code skill that generates structured, prioritized QA test checklists for
e-commerce storefronts — covering the full customer journey, not just cart and checkout.

Every run outputs **two formats**:

1. **Markdown table** — for quick review, PR comments, and documentation
2. **Google Apps Script (.gs)** — paste into Apps Script and run to populate a Google Sheet,
   one tab per module, with a `QA Tools` custom menu

Built for Shopify / Next.js storefronts, Indian D2C brands, and any product with heavy
offer, promo, discount, gift-card, or loyalty logic — the places where D2C bugs actually
cluster (coupon + offer + gift-card stacking conflicts, Buy Now bypassing the offer engine,
refund timelines, search relevance).

## Modules covered (13)

| Module | TC prefix | Examples of what gets tested |
|--------|-----------|------------------------------|
| Cart Logic | `CART` | BXAY, BXGY, tier pricing, offer stacking/exclusion, guest → logged-in cart merge |
| Add to Cart & Buy Now | `ATC` | Quick-add vs. variant-required add, Buy Now still applying offers, cart persistence |
| Checkout & Payments | `CHK` | COD / UPI / Card / Wallet / EMI / BNPL, gateway states, idempotency, webhook delay |
| Promo Codes & Coupon Stacking | `PROMO` | UTM vs. manual code priority, auto-apply, the full stacking matrix |
| Mystery Box / Freebies | `MYS` | Collection config, UTM-triggered reveal, OOS behaviour |
| Product Listing & PDP | `PDP` | Filters, sorting, variant → price update, OOS CTA, deep links |
| Search & Autocomplete | `SRCH` | Typo tolerance, synonyms, zero-result fallback, debounce |
| Login & Auth (incl. Guest Checkout) | `AUTH` | OTP flows, rate limiting, session persistence, guest → account merge |
| Order Management & Tracking | `ORD` | State machine transitions, cancellation window, GST invoice |
| Returns, Exchange & Refunds | `RET` | RMA window, partial returns, refund destination, timeline states |
| Gift Cards, Store Credit & Loyalty | `GC` | Partial redemption, expiry, points earn/redeem, refund impact |
| Shipping & Delivery | `SHIP` | Pincode serviceability, COD availability, free-shipping threshold, split shipments |
| Wishlist & Save for Later | `WISH` | Cross-device persistence, move to cart, guest wishlist merge |

Ask for one module and you get depth on that flow only. Ask for everything and you get all
13 with a summary table (~250–350 cases) and an offer to do a P1-only pass first.

## Test case schema

Each case carries: `TC_ID` · `Module` · `Sub-Feature` · `Test Case Title` · `Pre-conditions` ·
`Test Steps` · `Expected Result` · `Type` (Positive / Negative / Boundary / Edge) ·
`Priority` (P1–P3) · `Automation Status` (Automatable / Manual Only / Needs Data Setup).

Pre-conditions are required to be specific and expected results assertable — no
"works correctly".

## Installation

**Claude Code (project-level):**

```bash
git clone git@github.com:amitprajapati-afk/ecommerce-qa-checklist.git .claude/skills/ecommerce-qa-checklist
```

**Claude Code (user-level, available in every project):**

```bash
git clone git@github.com:amitprajapati-afk/ecommerce-qa-checklist.git ~/.claude/skills/ecommerce-qa-checklist
```

**Claude.ai:** zip the repo contents (`SKILL.md` + `references/`) and upload it under
Settings → Capabilities → Skills.

## Usage

The skill triggers automatically on requests like:

- "give me cart test cases"
- "test cases for buy now"
- "returns flow QA"
- "what should I test for checkout"
- "generate the full e-commerce test suite for this Shopify store"

Then paste the generated `.gs` into **Extensions → Apps Script** in a Google Sheet, run
`onOpen()` once, and use the `QA Tools` menu to populate each module tab.

## Repository layout

```
SKILL.md                                   # skill definition, module rules, output formats
references/
  offer-logic-patterns.md                  # BXGY / BXAY / tier pricing / coupon stacking matrix
  payments-and-checkout-patterns.md        # gateway states, BNPL, Buy Now, guest checkout, cart merge
  order-lifecycle-patterns.md              # order state machine, returns/refunds, shipping serviceability
  discovery-engagement-patterns.md         # search relevance, wishlist persistence, gift card / loyalty
```

The reference files are loaded on demand — the skill reads only the ones relevant to the
modules being generated, keeping context small.

## Apps Script guarantees

Generated scripts always: use `setValues()` with a 2D array (never per-cell loops),
null-check sheet existence, avoid `setFrozenColumns(0)`, give each module its own tab and
function, and register everything in a single `onOpen()` menu.

## Author

Amit Prajapati — QA Automation Engineer, [DevX AI Labs](https://devxlabs.ai)
GitHub: [@amitprajapati-afk](https://github.com/amitprajapati-afk)
