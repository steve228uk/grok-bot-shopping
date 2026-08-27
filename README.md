# Grok Bot shopping

Shopping skills for Grok Bot: merchant UCP/MCP, Shop catalog/Shop Pay (via `npx --yes @shopify/shop-cli`, Grok Bot cards), Shopify discount tests, Stripe Onelink checkout.

## Install

Paste this to Grok Bot:

```
Install Grok Bot shopping from https://github.com/steve228uk/grok-bot-shopping
Follow INSTALL.md.
```

Grok Bot will fetch the skills, skip a global Shop CLI install (always `npx --yes @shopify/shop-cli`), install libsecret/keytar if Shop auth cannot store tokens, offer Shop sign-in, tell you to turn on **Allow agent to pay for me** in [Shop connections](https://shop.app/account/settings/connections) (spend limit), optionally ask about Stripe Link CLI (US only), and ask whether to create a **Buyer** agent.

Nothing is required for UCP search or Onelink in the browser. Shop Pay and the Shop catalog always use:

```bash
npx --yes @shopify/shop-cli
```

Shop **sign-in** needs OS secret storage. On Linux that is libsecret (`libsecret-1-0` + `libsecret-1-dev`); on macOS, Keychain. Do not `npm install -g @shopify/shop-cli`. Do not use `--memory-store` for real login.

Grok Bot has **npm**, not pnpm. Do not install https://shop.app/SKILL.md as a Grok Bot skill. This pack wraps Shop's CLI via npx instead.

## Shop Pay agent pay (required for CLI complete)

Sign-in alone does **not** let Grok Bot finish payment in Shop CLI. For `checkout complete` you must:

1. Sign in to Shop (device-code during install, or later).
2. Open [Shop connections](https://shop.app/account/settings/connections).
3. Open the **Grok Bot** connection.
4. Toggle **Allow agent to pay for me**.
5. Set a **Spend limit** (Monthly / Weekly / Total) and an amount.

Verify from the agent:

```bash
npx --yes @shopify/shop-cli auth budget
```

`available: true` with a `remaining_amount` means the budget is live. Search and `checkout create` work without this. Completing still also needs the **merchant** to return a Shop Pay payment instrument; if instruments are empty, use the Finish in Shop / `continue_url` path instead.

## What's in the pack

| Path | What |
|---|---|
| `skills/merchant-agentic-commerce.md` | Discover llms.txt / /.well-known/ucp, search, cart, checkout. Cards: image + `[Name](url)`. |
| `skills/shop-catalog-for-grok-bot.md` | Shop CLI adapter via npx: catalog, Shop Pay, orders, tracking, returns. Same Grok Bot cards. |
| `skills/test-shopify-discount-codes.md` | Try promo codes on a Shopify Ajax cart. No checkout. |
| `skills/stripe-onelink-checkout.md` | Pay with Stripe Onelink in the browser (OTP paste if no iMessage). |
| `agents/buyer.md` | Persona for an optional Buyer teammate |
| `INSTALL.md` | The installer prompt Grok Bot follows |

## Platform profile

Shopify UCP MCP needs a public agent profile URL as `meta.ucp-agent.profile`. A working example:

https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-ucp-profile@main/ucp-agent-profile.json
