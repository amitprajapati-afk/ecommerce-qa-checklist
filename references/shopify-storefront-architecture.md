# Shopify Storefront Architecture: Theme vs. Headless

Read this when the Platform Profile says the store is on Shopify. The storefront architecture
decides *where* cart, offer, and session logic actually runs, so the same business rule fails
in different ways on a Liquid theme than on a headless build. Tag every case that only applies
to one architecture with `[Theme]` or `[Headless]` at the end of its Sub-Feature.

---

## Liquid Theme (Online Store 2.0)

On a theme, the storefront is rendered by Shopify, and most dynamic behavior goes through the
AJAX Cart API (`/cart/add.js`, `/cart/change.js`, `/cart.js`) plus theme JavaScript and app
embeds. Bugs cluster where theme JS and Shopify's server-side cart state disagree.

### Cart & Add to Cart `[Theme]`
- Cart drawer vs. cart page: add an item from PDP and verify the drawer, the header badge, and
  the `/cart` page all show the same lines, quantities, and totals — drawers often render from
  a stale JS copy instead of re-fetching `/cart.js`.
- Rapid double-click on Add to Cart: verify only one line/quantity increment is added, and the
  drawer doesn't show a duplicate line that disappears on refresh.
- Quantity change in the drawer: verify the discount/offer line recalculates after
  `/cart/change.js` returns, not before (premature UI update showing old discount).
- Line item properties (engraving, gift message, personalisation): verify they persist into
  checkout and onto the order in admin, and that two identical SKUs with different properties
  stay as separate lines.
- Variant switch on PDP: verify the URL `?variant=` parameter, price, compare-at price, stock
  state, and image all update together; deep-link with `?variant=` opens the right variant.

### Theme Customizer & Publishing `[Theme]`
- Test on the published theme, not only a preview/unpublished theme — app embeds, scripts,
  and settings are configured per theme and are often enabled only on one.
- Theme update or duplicate: verify app embeds and custom sections survive (common regression
  after a theme version upgrade).
- Section-level settings (announcement bar offers, free-shipping progress bar): verify the
  threshold shown in the bar matches the actual discount/shipping rule in admin.

### App Embeds & Blocks `[Theme]`
- Disable/remove an app embed in the customizer: verify the storefront degrades gracefully
  (no blank widget, no JS error that breaks Add to Cart).
- App script loads slowly or fails (throttle network / block the app domain): verify core
  purchase flow (PDP → cart → checkout) still works.

### Markets & Localization `[Theme]`
- If Shopify Markets is used: verify currency, price rounding, and localized URLs
  (e.g. `/en-in/`) carry through cart and checkout; switching market mid-session doesn't
  leave a cart priced in the old currency.

### Automation Notes `[Theme]`
- Liquid-rendered markup has unstable selectors; add stable `data-testid` hooks in Liquid
  (the `add-testids` skill supports Liquid) and mark cases that depend on them as
  `Needs Data Setup` until hooks exist.
- AJAX Cart API calls can be asserted directly with Playwright network interception —
  prefer asserting `/cart.js` JSON over scraping drawer text for totals.

---

## Headless (Hydrogen, Next.js, or other custom frontend)

On a headless build, the frontend talks to the Storefront API (GraphQL) and hands the shopper
to Shopify checkout through the cart's `checkoutUrl`. Bugs cluster in caching, the cart
handoff, and features that apps only provide as theme widgets.

### Cart & Checkout Handoff `[Headless]`
- Cart created via Storefront API: verify cart ID persists across refresh, tab close, and
  login (stored in cookie/local storage) and doesn't silently create a new empty cart.
- Discount code applied on the headless cart: verify it's applied on the Shopify cart object
  (Storefront API), so it's still present after redirect to `checkoutUrl` — not just
  calculated in frontend state.
- Checkout handoff: verify line items, quantities, attributes, and buyer identity
  (logged-in email/phone, market/country) match exactly between the headless cart and the
  checkout page.
- Return from checkout (back button or "return to store"): verify the headless cart reflects
  the true Shopify cart state, including after a completed order (cart should be cleared).

### Caching & Data Freshness `[Headless]`
- Price or inventory change in admin: verify PDP/PLP reflect it within the expected
  revalidation window; verify a cached "in stock" PDP can't add an OOS item that then fails
  only at checkout.
- Offer start/end time: verify a cached page doesn't show an expired offer badge or price.
- Product unpublished/deleted: verify its cached PDP returns a proper 404 or redirect, and
  that it can't be added to cart.

### Auth & Customer Accounts `[Headless]`
- Customer login via the headless auth flow: verify session carries into checkout (prefilled
  details, saved addresses) and back out (order history visible after purchase).
- Logout: verify both the headless session and the checkout session are cleared.

### App Features on Headless `[Headless]`
- Apps built as theme widgets (reviews, loyalty, wishlist, search) don't render
  automatically on headless — verify each one is integrated through its API/SDK and that
  data written on headless (a review, a wishlist item, redeemed points) appears in the app's
  admin and vice versa.

### API Behavior `[Headless]`
- Storefront API errors or throttling (simulate via network mocking): verify user-facing
  error states, not blank pages or infinite spinners.
- Verify SEO-relevant output is server-rendered (product title, price, structured data) if
  the build claims SSR.

---

## When Architecture Is Unknown

Generate the shared cases normally, then add the `[Theme]` and `[Headless]` cart/handoff
cases for both, and note in the summary that the user should delete the non-applicable set
once confirmed.
