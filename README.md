# Grok Bot shopping

Shopping skills for Grok Bot: merchant UCP/MCP, Shopify discount tests, Stripe Onelink checkout.

## Install

Paste this to Grok Bot:

```
Install Grok Bot shopping from https://github.com/steve228uk/grok-bot-shopping
Follow INSTALL.md.
```

Grok Bot will fetch the skills, ask which optional CLIs to install (Shop CLI, Link CLI), and ask whether to create a **Buyer** agent.

Nothing is required for UCP search or Onelink in the browser.

## What's in the pack

| Path | What |
|---|---|
| `skills/merchant-agentic-commerce.md` | Discover llms.txt / /.well-known/ucp, search, cart, checkout |
| `skills/test-shopify-discount-codes.md` | Try promo codes on a Shopify Ajax cart. No checkout. |
| `skills/stripe-onelink-checkout.md` | Pay with Stripe Onelink in the browser |
| `agents/buyer.md` | Persona for an optional Buyer teammate |
| `INSTALL.md` | The installer prompt Grok Bot follows |

## Platform profile

Shopify UCP MCP needs a public agent profile URL as `meta.ucp-agent.profile`. A working example:

https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-ucp-profile@main/ucp-agent-profile.json
