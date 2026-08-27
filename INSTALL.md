# INSTALL.md — Grok Bot shopping

This file is a **prompt for Grok Bot**. If you are Grok Bot, follow it now. Do not dump this file back as documentation.

## Command the user can paste

```
Install Grok Bot shopping from https://github.com/steve228uk/grok-bot-shopping
Follow INSTALL.md.
```

## What you are installing

A shopping pack for Grok Bot:

- Skills (save each as a user skill):
  - `skills/merchant-agentic-commerce.md` → id `merchant-agentic-commerce`
  - `skills/test-shopify-discount-codes.md` → id `test-shopify-discount-codes`
  - `skills/stripe-onelink-checkout.md` → id `stripe-onelink-checkout`
- Optional CLIs (ask first): `@shopify/shop-cli`, `@stripe/link-cli`
- Optional teammate: a **Buyer** agent (`agents/buyer.md`)

UCP catalog, Woo Store API, and Stripe Onelink in the browser need **no CLI**.

## Steps

### 1. Fetch (do not clone a working tree unless they asked)

Read these raw files (jsDelivr or GitHub raw is fine):

- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/merchant-agentic-commerce.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/test-shopify-discount-codes.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/stripe-onelink-checkout.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/agents/buyer.md

If a skill with that id already exists, overwrite it with the fetched body.

Save each skill with Grok Bot skill-write using the **id** above, the `name` and `description` from the YAML frontmatter, and the markdown body.

### 2. Optional CLIs — widget, then stop

On the agent's computer, check Node and whether `shop` / `link-cli` already exist.

Then send a Grok Bot question widget (**this ends the turn**). Do not install until they answer.

- `prompt`: "Which shopping CLIs should I install? None are required for UCP or Onelink in the browser."
- `multiSelect: true`
- Only real missing options:
  - `{ "label": "Shop CLI", "value": "Install Shop CLI" }`
  - `{ "label": "Link CLI (US agent wallet)", "value": "Install Link CLI" }`
- If both are already installed, skip the widget and say so.

Dismiss or an empty pick means skip. If they picked any:

```bash
npm install --global @shopify/shop-cli   # only if they picked Shop CLI
npm install --global @stripe/link-cli    # only if they picked Link CLI
```

Do not run `shop auth` or `link-cli auth login` during install.

### 3. Buyer agent — widget, then stop

If a teammate named Buyer already exists, skip this and say it is already there.

Otherwise send a question widget (**ends the turn**):

- `prompt`: "Want me to create a Buyer agent to own shopping?"
- Options:
  - `{ "label": "Create Buyer", "value": "Create a Buyer agent", "style": "primary" }`
  - `{ "label": "Skip", "value": "Don't create a Buyer agent" }`

If they confirm, `CreateAgent` with name `Buyer` and the description from `agents/buyer.md` (use the user's name if you know it). Then stop. Do not message Buyer unless they asked.

### 4. Done

Tell them:

- Skills installed (names).
- Which CLIs landed (or that you skipped).
- Whether Buyer was created.
- They can shop by asking you or @Buyer. Confirm-before-pay still applies.

## Guardrails

- Never auto-install CLIs.
- Never auto-create Buyer.
- Never collect PANs or ask for passwords in chat.
- Confirm-before-pay stays a widget.
