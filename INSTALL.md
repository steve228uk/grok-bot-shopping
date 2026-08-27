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

UCP catalog, Woo Store API, and Stripe Onelink in the browser need **no CLI**. Shop Pay / Shop catalog use `npx --yes @shopify/shop-cli` (or Shop's direct catalog MCP if npx is blocked). Shop catalog is an **augment** (the merchant skill covers Woo, UCP, and other non-Shop origins).

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

Catalog (`search` / `catalog`) must identify as Grok Bot. Pass this global flag BEFORE the subcommand:

```bash
npx --yes @shopify/shop-cli --profile-url "https://cdn.jsdelivr.net/gh/steve228uk/grok-bot-ucp-profile@main/ucp-agent-profile.json" search "…"
```

Pass `--country XX` only after buyer country is set (next steps). Never invent country; never assume GB.

Shop catalog is an **augment**. The merchant skill covers Woo, UCP, HTML, and other non-Shop origins; do not treat Shop as the only search.

`--device-name "Grok Bot"` is only for `auth device-code` (Shop Connections). Do not use the agent name or hostname.

Grok Bot's computer has **npm**, not pnpm. `--yes` skips the npx prompt. Do not run `auth login` / `auth device-code` in this step.

If npx is blocked, follow https://shop.app/references/catalog-mcp.md and https://shop.app/references/direct-api.md from the Shop catalog skill.

### 3. Secret store (libsecret) — Linux CLI, no GUI keyring dialog

Run:

```bash
npx --yes @shopify/shop-cli auth status
```

Good: JSON like `{"authenticated": false}` or `true`. Bad: `OS secret storage is unavailable. Install/build keytar or run in an environment with macOS Keychain support.`

`--memory-store` is tests only. Do not use it for real sign-in (tokens die when the npx process exits).

A global `npm install -g keytar` is usually **not** enough for npx Shop CLI. On Linux, Shop's bundled keytar needs **libsecret** plus an **unlocked** GNOME keyring.

**Do not** hand the user a GTK "Choose password for new keyring" dialog. That window freezes in Grok Bot (no clicks, cannot close). Unlock from the terminal instead.

If auth status failed on secret storage, or `auth device-code` hangs, on Linux:

1. Widget first (**ends the turn**) unless they already asked you to install it:
   - `prompt`: "Shop sign-in needs libsecret and an unlocked keyring on this computer. Install that?"
   - `{ "label": "Install libsecret", "value": "Install libsecret for Shop auth", "style": "primary" }`
   - `{ "label": "Skip", "value": "Skip Shop secret store" }`
2. If they confirm:

```bash
sudo apt-get update
sudo apt-get install -y libsecret-1-0 libsecret-1-dev libsecret-tools pkg-config build-essential
# If a gcr-prompter / keyring password window is already up, kill it:
pkill -f gcr-prompter || true
echo -n '' | gnome-keyring-daemon --unlock
npx --yes @shopify/shop-cli auth status
```

The empty `echo -n ''` unlocks (or creates) the default keyring with no password. That is the intended path on Grok Bot's Linux computer. Do **not** `request_box_help` for this.

On macOS, Keychain should already work; if it does not, say so instead of apt.

Retry `auth status`. It should no longer mention keytar. Then continue to buyer country.

If they skip, continue. Shop search stays unsigned.

### 4. Buyer country — widget, then stop

Run:

```bash
npx --yes @shopify/shop-cli config show
```

If `country` is already a 2-letter code, skip this widget and say so.

Shop CLI `--country` is **optional**. Omitting it makes the CLI default **US**. A signed-in Shop account does **not** set it (`auth status` has no country; `config show` is `{"country": null}` until `config set-country`).

Never invent a country. Never assume GB. Never infer it from the model, a name, or a phone prefix.

If country is not set, send a Grok Bot question widget (**this ends the turn**). `allowCustom`: true.

- `prompt`: "Which country should Shop use for prices and shipping?"
- `helpText`: "If yours isn't listed, type a two-letter ISO country code (for example IE)."
- Options (exactly these five):
  - `{ "label": "United States (US)", "value": "Set Shop country to US" }`
  - `{ "label": "United Kingdom (GB)", "value": "Set Shop country to GB" }`
  - `{ "label": "Canada (CA)", "value": "Set Shop country to CA" }`
  - `{ "label": "Australia (AU)", "value": "Set Shop country to AU" }`
  - `{ "label": "Germany (DE)", "value": "Set Shop country to DE" }`

Custom answers: accept only a 2-letter ISO country code; uppercase it. If invalid, send the widget again.

Then:

```bash
npx --yes @shopify/shop-cli config set-country XX
```

After that, pass `--country XX` on catalog calls, plus matching `--currency` / `--ships-to` when you know them.

### 5. Shop sign-in — widget, then stop

If `auth status` is signed-out (and secret storage works), offer sign-in once. Widget (**ends the turn**):

- `prompt`: "Want to sign in to Shop for shipping rates, Shop Pay, and order history?"
- `{ "label": "Sign in to Shop", "value": "I'll sign in to Shop", "style": "primary" }`
- `{ "label": "Continue unsigned", "value": "Continue without Shop sign-in" }`

If they sign in:

1. If `auth device-code` hangs, you likely hit the GUI keyring dialog. Kill `gcr-prompter` and the hung `shop` process, run `echo -n '' | gnome-keyring-daemon --unlock`, retry.
2. `npx --yes @shopify/shop-cli auth device-code --device-name "Grok Bot"`
3. Send `verification_uri_complete` as a plain `[Sign in to Shop](url)` (**no** wrapping `**`).
4. Widget: they've signed in / cancel. **STOP**.
5. After they confirm, `npx --yes @shopify/shop-cli auth poll` until not `pending`. Recheck `auth status`.

If they continue unsigned, skip poll.

### 6. Optional Link CLI — widget, then stop

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

### 7. Buyer agent — widget, then stop

If a teammate named Buyer already exists, skip this and say it is already there.

Otherwise send a question widget (**ends the turn**):

- `prompt`: "Want me to create a Buyer agent to own shopping?"
- Options:
  - `{ "label": "Create Buyer", "value": "Create a Buyer agent", "style": "primary" }`
  - `{ "label": "Skip", "value": "Don't create a Buyer agent" }`

If they confirm, `CreateAgent` with name `Buyer` and the description from `agents/buyer.md` (use the user's name if you know it). Then stop. Do not message Buyer unless they asked.

### 8. Done

Tell them:

- Skills installed (names).
- Shop CLI is invoked with `npx --yes @shopify/shop-cli` (not installed globally).
- Whether libsecret + CLI keyring unlock landed (or that you skipped).
- Which buyer country was set (or that it was already set).
- Whether they signed in to Shop (or unsigned).
- Whether Link CLI landed (or that you skipped).
- Whether Buyer was created.
- They can shop by asking you or @Buyer. Confirm-before-pay still applies.

## Guardrails

- Never `npm install -g @shopify/shop-cli`. Always `npx --yes @shopify/shop-cli`.
- Never use `--memory-store` for real Shop sign-in.
- Never hand the user a GNOME "Choose password for new keyring" dialog. Unlock with `echo -n '' | gnome-keyring-daemon --unlock`.
- Never auto-install libsecret or Link CLI (widget first, unless they already asked).
- Never auto-create Buyer.
- Never install https://shop.app/SKILL.md as a Grok Bot skill.
- Never collect PANs or ask for passwords in chat.
- Confirm-before-pay stays a widget.
- Grok Bot has npm, not pnpm. Do not run pnpm install commands.
- Always --profile-url with the Grok Bot UCP profile on search and catalog.
- Never invent country. Ask at setup. Do not assume GB. A Shop account does not set country.
- Shop is an augment (merchant skill covers Woo/etc).
