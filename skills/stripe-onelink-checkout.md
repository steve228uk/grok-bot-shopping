---
name: Stripe Onelink checkout
description: >-
  Use this when a checkout uses Stripe Onelink (Link) in the browser — SMS OTP
  from Onelink, then tappable widgets for address/card options, then confirm
  with UI buttons. Preferred UK pay path; Google Pay in box Chrome is the
  backup. Skip US-only link-cli. Allow the user to paste the Onelink OTP in chat
  when iMessage is not installed.
---
# Stripe Onelink (Link) checkout

Use when a merchant checkout presents **Stripe Onelink / Link** (autofill email → SMS code → saved address & card). SMS sender label is often **Onelink**.

This is the **preferred UK pay path** in box Chrome. Stripe `link-cli` agent credentials are US-only — skip them unless the buyer is on the US agent-wallet path. If Onelink is not on the checkout, try **Google Pay** in the same Chrome session. `request_box_help` only for bank/biometric/captcha.

## Principles

- Prefer completing Onelink **yourself** in the browser (box `browserUse` / `computerUse`): enter OTP, then click the chosen address/card confirm buttons.
- **OTP source (pick the first that works):**
  1. If [iMessage on Mac](sand-workflow:imessage-on-mac) is installed and can read SMS, read the Onelink code from Messages. Do not ask them to paste it.
  2. Otherwise **ask them to paste the Onelink code into this chat**. Most installs will not have iMessage. Wait for the paste, use it in the Onelink UI, then do not repeat the digits in later messages.
- When Onelink shows a **list of options** (addresses, cards, payment methods), present them as a Grok Bot **question widget** (tappable options; `multiSelect` only if several can apply). Do not silently choose. Do not dump a bullet menu.
- Use `request_box_help` only for a step you truly cannot do (e.g. bank app push, captcha, biometric that only they can approve).
- Do not invent card numbers or ask for card details in chat.

## Prerequisites

- Box browser on the checkout page (or resume there).
- iMessage is **optional**. If the user's Mac has [iMessage on Mac](sand-workflow:imessage-on-mac) (`imsg` via ExternalShell) with Full Disk Access, prefer that for OTP. If `imsg` is missing, FDA is denied, or this is not a Mac, skip it and ask them to paste the code.
- Messages Automation permission is only needed if you also *send* or *delete* iMessages.

## Flow

### 1. Start Onelink on the checkout

1. Open the merchant payment step.
2. If Onelink/Link appears, enter the user's checkout **email** (the one tied to their Onelink account).
3. Trigger continue / verify so Onelink sends the SMS OTP.
4. Note the time you triggered it (use the newest code after that).

### 2. Get the OTP

**If iMessage is available**, on the user's Mac (ExternalShell):

```bash
imsg chats --limit 40 --json
imsg history --chat-id <onelink-chat-id> --limit 5 --json
# Or: imsg search "Onelink verification" --json
```

Parse a message like: `###### is your Onelink verification code` (6 digits). Keep `guid` and `chat_guid` for delete.

Rules when reading from iMessage:

- Use the **newest** code created **after** you triggered verification.
- If nothing arrives within ~30–60s, wait briefly and re-check history once.
- Enter the code in the Onelink UI yourself.
- **After the code is accepted**, delete that OTP message (`imsg delete-message`) per the iMessage skill.

**If iMessage is not available** (missing CLI, no Mac, no FDA, or the read failed):

1. Tell them Onelink just texted a code and ask them to **paste it here**.
2. Use the pasted digits in the Onelink UI. Do not echo the code back.
3. Do not stall on installing iMessage during a live checkout.

### 3. Widget the address list, then confirm in the UI

After OTP succeeds, Onelink may show saved shipping/billing address(es).

1. Read every option on screen (label, full address, any default badge).
2. **Question widget** with those real addresses (do not invent). Wait. Do not auto-pick unless they already named the address for this order.
3. When they choose, **click the UI button** for that address / Continue.
4. If the UI allows editing and they need a change you can make, do it; if not, stop and ask.

### 4. Widget the card list, then confirm payment in the UI

1. Read every payment option Onelink shows (brand, last4, expiry if shown, bank labels).
2. **Question widget** for which card/method — do not silently select.
3. After they choose, **click Pay / Confirm / Place order** for that method (still only if they confirmed this order).
4. If a further 3DS / bank challenge needs the user (app approve, biometric), hand over with `request_box_help` (reason `payment` or `auth`) with a one-line instruction, then continue after they hand back.
5. Wait for order confirmation; capture order id / total / delivery if shown.

### 5. Report back

Tell the user: merchant, order id (if any), amount, address used (short), card last4 if shown, and that Onelink OTP was entered (do not dump the code). If it came from iMessage, confirm that OTP message was deleted.

## Failure / edge cases

| Symptom | What to do |
|---|---|
| No Onelink SMS | Confirm email/phone on the form; retry send code; if iMessage is live, re-check history; else ask them to paste |
| Code rejected / expired | Request a new code; take the newest SMS or paste only; delete stale used/superseded OTPs if iMessage |
| Option list unclear | Screenshot; widget only the options you can actually see |
| Pay button disabled | Check required fields / terms checkbox; screenshot and fix |
| `imsg` missing | Ask them to paste the Onelink code in chat. Do not block checkout on installing iMessage |
| User must approve bank push | `request_box_help` — do not ask for passwords or card numbers in chat |
| Onelink not on this checkout | Try Google Pay in the same box Chrome; else `request_box_help` |

## Do not

- Repeat OTP digits, full card numbers, or CVCs in later chat messages.
- Silently pick among Onelink address or card options — **always widget the list**.
- Skip confirmation buttons after the user has chosen.
- Use `link-cli` unless the buyer is on the US agent-wallet path (US-only).
- Block a live checkout to install iMessage.
