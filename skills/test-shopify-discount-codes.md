---
name: Test Shopify discount codes
description: >-
  Use this when testing many Shopify discount or promo codes against a cart and
  reporting which codes change the total. Do not checkout. Present code/product
  choices as Grok Bot widgets, not prose menus.
---
# Test Shopify discount codes

Use when you have a Shopify store URL, a cart (or product to add), and a list of discount/promo codes to try. Goal: report which codes change the cart total. **Never checkout** unless the user has confirmed that specific order.

This is a **customer-side** flow (Ajax cart / Storefront cart / `/discount/CODE`). You cannot list a shop’s valid codes via Admin API without the merchant’s token. Do not build or install a Chrome extension for this.

## Choices: Grok Bot widgets

Whenever the user must pick (which codes to try, which hit to apply, variant if the cart is ambiguous):

- Send a **Grok Bot question widget**, not a prose/bullet menu.
- `prompt` is a natural question (e.g. "Which of these codes should I try?").
- Each option `value` reads like a reply they would send.
- Only **real, verified** codes/products from their list or the cart. Never invent codes.
- `multiSelect: true` when several codes may apply.
- If you will place an order after a hit, confirm-before-pay is a widget: primary confirm, danger cancel.
- Max 6 options; if the code list is longer, widget a verified subset or ask them to narrow.

## Inputs

- Store URL (any `*.myshopify.com` or custom-domain Shopify storefront)
- Cart: product URL(s) + quantities, or an existing cart/checkout session
- Code list (plain list; skip empties and duplicates)
- Optional: shipping country if the store needs it for the total to be meaningful

## Principles

- Try codes programmatically, not by clicking the discount field one-by-one.
- One cart, one code at a time (most Shopify shops allow a single discount).
- **Go slow.** Default **4–6 seconds** between code attempts (not 1s). After building the cart, wait a few seconds before the first apply. Cap **~8–10 codes** unless the user asked for more. Stop at the first useful saving if they only needed “best of this list.”
- Prefer the **box Chrome cookie jar** (browserUse on the store) over a blank `curl` session. Cloudflare is much likelier to 429 a fresh datacenter IP.
- Only test codes the user supplied or asked you to try on a store they are buying from. Do not scrape/spray huge public dumps.
- **Checkout is out of scope.** Report results and wait for an explicit confirm on that order.

## Flow

### 1. Confirm it is Shopify

Open the store (or product) page. Look for `Shopify.shop`, `window.Shopify`, `cdn.shopify.com`, or `/cart.js` responding with JSON. If it is not Shopify, stop and say so.

### 2. Build a cart

Prefer the Ajax cart **in the same session** you will test codes:

1. `POST /cart/clear.js` then add the requested variant(s) via `POST /cart/add.js` (variant id from the product page / `/products/<handle>.js`).
2. Read `GET /cart.js` and record **baseline** `total_price`, `items_subtotal_price`, item titles, and `token`.
3. Pause ~3s before applying the first code.

If Ajax cart is blocked, use the Storefront Cart GraphQL API (step 4) to `cartCreate` with merchandise lines instead.

### 3. Preferred: Ajax `cart/update.js` (since May 2025)

Loop, one code at a time, **4–6s apart**:

```
POST /cart/update.js
Content-Type: application/json
{"discount":"CODE"}
```

Then read the JSON (or `GET /cart.js`) and compare `total_price`, `total_discount`, `discount_codes[].applicable`, and `cart_level_discount_applications` to baseline.

Clear before the next try (then wait again):

```
POST /cart/update.js
{"discount":""}
```

Comma-separated codes in `discount` are allowed by Shopify but most stores will not stack — still test one at a time.

### 3b. HTTP 429 / Cloudflare challenge → box browser

If any cart/discount request returns **429**, HTML challenge, or `cf-mitigated: challenge`:

1. **Stop the curl/Ajax loop immediately.** Do not retry the same origin from a blank session.
2. Wait ~30s (do not hammer).
3. **Fall back to box Chrome** (`browserUse` on the store, existing cookies if any). Rebuild the cart in that browser if needed.
4. Continue remaining codes there, still one at a time and slowly (type/apply, wait for total, clear). If the browser also hits a challenge, **stop** and report how far you got — then `request_box_help` only if a human CF/captcha is required to continue a purchase the user already confirmed.
5. Tell the user you hit a rate limit and switched to the browser.

Do not keep looping curl through a bot wall.

### 4. Optional: Storefront token GraphQL

Search page HTML / theme JS for a Storefront access token (`storefrontAccessToken`, `storefront-api`, `Shopify.storefront`). If found, use GraphQL at `/api/<version>/graphql.json`. Same slow cadence; same 429 → browser fallback.

```graphql
mutation cartDiscountCodesUpdate($cartId: ID!, $discountCodes: [String!]) {
  cartDiscountCodesUpdate(cartId: $cartId, discountCodes: $discountCodes) {
    cart {
      id
      cost { totalAmount { amount currencyCode } subtotalAmount { amount currencyCode } }
      discountCodes { code applicable }
      discountAllocations { discountedAmount { amount } }
    }
    userErrors { field message }
  }
}
```

- Apply `[code]`, read whether `applicable` is true and whether total dropped vs baseline.
- Then apply `[]` so the next try is clean.

### 5. Fallback: `/discount/CODE`

If Ajax update is unavailable (and you are not already 429’d):

1. With the same cart cookies, `GET {store}/discount/{code}` (follow redirects).
2. `GET /cart.js` again. Compare totals as above.
3. Clear via `POST /cart/update.js` `{"discount":""}`, or `/cart/clear.js` and re-add items.

### 6. What to treat as a hit

A hit is **the cart total lower than baseline**, or a non-empty discount application with `applicable: true`. `applicable: false` is a miss even if the code is echoed on the cart.

Ignore cosmetic “code accepted” UI if the total did not change.

### 7. Report

Return a short table: code, hit/miss, saving (amount + currency), any restriction (min spend, product, first order). Highlight the best saving. If they should pick which hit to keep, use a widget. Do not place the order. Mention if you had to fall back to the browser after a 429.

## Guardrails

| Do | Don't |
|---|---|
| Delay **4–6s** between codes; cap ~8–10 unless asked | Burst codes; 1s loops |
| Reuse one cart session (box browser cookies if present) | Invent Admin API calls or guess a merchant token |
| On 429: stop curl, wait, **fall back to box browser** | Keep retrying curl through Cloudflare |
| Keep checkout confirmation with the shopping agent / user | Auto-apply a code at payment after this test unless asked |

## Failure / edge cases

- **HTTP 429 / CF challenge:** fall back to box browser (above). If the browser is challenged too, stop and report.
- **Checkout-only discounts:** some codes only apply on `/checkout`. Try Ajax/Storefront first; if totals never move, say the shop may require checkout-step application and stop short of paying.
- **Shipping codes:** may need a shipping address before the total changes. Ask country/postcode with a widget of known addresses if you have them, rather than guessing.
- **Plus scripts / custom checkout:** Ajax + Storefront may both fail. Report that and stop; do not click through payment.

## Live test note (Pet Supermarket, Aug 2026)

`POST /cart/update.js {"discount":"CODE"}` returned JSON with `discount_codes: [{code, applicable: false}]` for the first codes, then Cloudflare 429 from a **fast** blank curl session. Slow cadence + browser fallback is the fix.
