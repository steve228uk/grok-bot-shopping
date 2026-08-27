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
  - `skills/shop-catalog-for-grok-bot.md` → id `shop-catalog-for-grok-bot`
  - `skills/test-shopify-discount-codes.md` → id `test-shopify-discount-codes`
  - `skills/stripe-onelink-checkout.md` → id `stripe-onelink-checkout`
- Shop CLI is **not installed globally**. Always run `npx --yes @shopify/shop-cli` (same subcommands). Do not `npm install -g`.
- Shop **sign-in** needs OS secret storage (keytar / libsecret on Linux, Keychain on macOS). Search works unsigned without it.
- Optional CLI (ask first): `@stripe/link-cli` (US agent wallet only)
- Optional teammate: a **Buyer** agent (`agents/buyer.md`)

Do **not** install https://shop.app/SKILL.md as a Grok Bot skill (wrong card format). The Shop catalog skill wraps Shop's CLI via npx instead.

UCP catalog, Woo Store API, and Stripe Onelink in the browser need **no CLI**. Shop Pay / Shop catalog use `npx --yes @shopify/shop-cli` (or Shop's direct catalog MCP if npx is blocked).

## Steps

### 1. Fetch (do not clone a working tree unless they asked)

Read these raw files (jsDelivr or GitHub raw is fine):

- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/merchant-agentic-commerce.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/shop-catalog-for-grok-bot.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/test-shopify-discount-codes.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/skills/stripe-onelink-checkout.md
- https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-shopping@main/agents/buyer.md

If a skill with that id already exists, overwrite it with the fetched body.

Save each skill with Grok Bot skill-write using the **id** above, the `name` and `description` from the YAML frontmatter, and the markdown body.

### 2. Shop CLI — no global install

Do not install `@shopify/shop-cli` globally. Do not check whether `shop` is on PATH. Every Shop command is:

```bash
npx --yes @shopify/shop-cli <subcommand> …
```

Grok Bot's computer has **npm**, not pnpm. `--yes` skips the npx prompt. Do not run `auth login` / `auth device-code` in this step.

If npx is blocked, follow https://shop.app/references/catalog-mcp.md and https://shop.app/references/direct-api.md from the Shop catalog skill.

### 3. Secret store (keytar) — widget if broken

Run:

```bash
npx --yes @shopify/shop-cli auth status
```

Good: JSON like `{"authenticated": false}` or `true`. Bad: `OS secret storage is unavailable. Install/build keytar or run in an environment with macOS Keychain support.`

`--memory-store` is tests only. Do not use it for real sign-in (tokens die when the npx process exits).

A global `npm install -g keytar` is usually **not** enough for npx Shop CLI. On Linux, Shop's bundled keytar needs **libsecret**.

If auth status failed on secret storage, send a widget (**ends the turn**):

- `prompt`: "Shop sign-in needs OS secret storage (keytar). Install libsecret so tokens can be saved?"
- `{ "label": "Install libsecret", "value": "Install libsecret for Shop auth", "style": "primary" }`
- `{ "label": "Skip", "value": "Skip Shop secret store" }`

If they confirm, on Linux:

```bash
sudo apt-get update
sudo apt-get install -y libsecret-1-0 libsecret-1-dev libsecret-tools pkg-config build-essential
npx --yes @shopify/shop-cli auth status
```

On macOS, Keychain should already work; if it does not, say so instead of apt.

Retry `auth status`. It should no longer mention keytar. Do not start device-code yet.

If they skip, continue. Shop search stays unsigned.

### 4. Shop sign-in — widget, then stop

If `auth status` is signed-out (and secret storage works), offer sign-in once. Widget (**ends the turn**):

- `prompt`: "Want to sign in to Shop for shipping rates, Shop Pay, and order history?"
- `{ "label": "Sign in to Shop", "value": "I'll sign in to Shop", "style": "primary" }`
- `{ "label": "Continue unsigned", "value": "Continue without Shop sign-in" }`

If they sign in: `npx --yes @shopify/shop-cli auth device-code --device-name "<agent> - <host>"`, send `verification_uri_complete` as a plain `[Sign in to Shop](url)`, **STOP**, then `npx --yes @shopify/shop-cli auth poll` until not `pending`. Recheck `auth status`.

If they continue unsigned, skip poll.

### 5. Optional Link CLI — widget, then stop

On the agent's computer, check whether `link-cli` already exists.

Grok Bot's computer has **npm**, not pnpm. Always use npm.

If `link-cli` is already installed, skip this widget and say so.

Otherwise send a Grok Bot question widget (**this ends the turn**). Do not install until they answer.

- `prompt`: "Install Stripe Link CLI? It is US-only for agent wallets. Skip it in the UK. Shop catalog uses npx and does not need a global Shop CLI."
- Options:
  - `{ "label": "Install Link CLI", "value": "Install Link CLI" }`
  - `{ "label": "Skip", "value": "Don't install Link CLI" }`

If they picked Link CLI:

```bash
npm install --global @stripe/link-cli
```

Do not run `link-cli auth login` during install.

### 6. Buyer agent — widget, then stop

If a teammate named Buyer already exists, skip this and say it is already there.

Otherwise send a question widget (**ends the turn**):

- `prompt`: "Want me to create a Buyer agent to own shopping?"
- Options:
  - `{ "label": "Create Buyer", "value": "Create a Buyer agent", "style": "primary" }`
  - `{ "label": "Skip", "value": "Don't create a Buyer agent" }`

If they confirm, `CreateAgent` with name `Buyer` and the description from `agents/buyer.md` (use the user's name if you know it). Then stop. Do not message Buyer unless they asked.

### 7. Done

Tell them:

- Skills installed (names).
- Shop CLI is invoked with `npx --yes @shopify/shop-cli` (not installed globally).
- Whether libsecret/keytar landed (or that you skipped).
- Whether they signed in to Shop (or unsigned).
- Whether Link CLI landed (or that you skipped).
- Whether Buyer was created.
- They can shop by asking you or @Buyer. Confirm-before-pay still applies.

## Guardrails

- Never `npm install -g @shopify/shop-cli`. Always `npx --yes @shopify/shop-cli`.
- Never use `--memory-store` for real Shop sign-in.
- Never auto-install libsecret/keytar or Link CLI.
- Never auto-create Buyer.
- Never install https://shop.app/SKILL.md as a Grok Bot skill.
- Never collect PANs or ask for passwords in chat.
- Confirm-before-pay stays a widget.
- Grok Bot has npm, not pnpm. Do not run pnpm install commands.
