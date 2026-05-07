# Virtual Cards

Request, list, get checkout credentials for, and inspect virtual cards on the agent wallet.

## What virtual cards are

Virtual cards are temporary, scoped payment cards generated from the user's saved card on file using **Visa Intelligent Commerce (VIC)** and **Mastercard Agent Pay** — agent-payment protocols that issue short-lived, intent-bound credentials specifically for AI agents. They work like disposable debit cards with a fixed spending limit.

Key properties:

- **Privacy-preserving:** The user's real card details (number, CVC, expiry) are never shared with the agent. Every value the agent sees is a protocol-issued artifact, not the user's saved card.
- **Scoped balance:** Each virtual card has a hard spending cap set at creation time. The card declines further charges once the cap is reached.
- **Human approval required:** Creating a virtual card always requires user approval via a link. The agent cannot create or fund a card without explicit human consent.

**Naming:** The API calls these _order intents_; the CLI exposes them as _virtual cards_. Each card has a stable id: `orderIntentId`. For `lobstercash cards reveal`, pass that value as `--card-id`.

---

## Requesting a new card

See [cards-request.md](cards-request.md). Conceptually this is a **card permission** — the user grants the agent a capped, purpose-scoped right to charge.

---

## Listing existing cards

This command shows card metadata (description, limit, phase, and card ID) — not payment credentials. To get the credentials needed for checkout, use `cards reveal` (see below) with the `card-id` from this output.

### Command

```bash
lobstercash cards list
```

### Reading the output

One line per card:

```
  <description>  $<amount> <currency> limit  [<phase>]  card-id=<orderIntentId>
```

Use `card-id` as `--card-id` when running `lobstercash cards reveal`. In OpenClaw, the same value is `orderIntentId` on each item in `lobster_order_intents` → `details.orderIntents`.

Possible phase values:

- `active` — card is ready to use; credentials can be revealed for checkout
- `requires-payment-method` — no valid card on file; the user must add or update their payment method at lobster.cash/dashboard before this card can activate
- `requires-verification` — the user's bank requires additional authentication (e.g. 3D Secure); they must complete verification in the browser at the approval link
- `expired` — card is no longer valid; the user needs to request a new one

### When to use

- After the user approves a card request — to confirm the card is `active` and get its `card-id`
- When the user asks "do I have any cards"
- To check the status of a specific card
- **Before creating a new card for a purchase** — always list first and check whether an existing card can be reused (see "Reusing an existing card" below). This is Step 3 of the buy flow in `SKILL.md`.

### Reporting to the user

List only `active` cards unless the user asks for all.
Say: "You have [n] active card(s): [description] with a $[amount] limit."

### Reusing an existing card

When you're about to create a new card for a purchase, run `cards list` first and check whether one already covers the purchase. Reusing a card avoids spamming the user with another approval link.

> **Note:** Cards are single-use today (subscription cards coming soon). A card you already charged successfully cannot be reused — request a new one. The `period` rule below is forward-looking for when subscriptions ship.

Each item carries these mandate fields:

- `phase` — only `active` is usable.
- `mandates[type=maxAmount].value` — the spending cap.
- `mandates[type=maxAmount].details.currency` — currency (default `USD`).
- `mandates[type=maxAmount].details.period` — currently always absent (single-use). Will be `weekly | monthly | yearly` when subscriptions launch.
- `mandates[type=description].value` — what the user approved it for.
- `mandates[type=prompt].value` — richer original natural-language request (when present).
- `mandates[type=consumer].details.email` — consumer email scope.

Do **not** match on:

- **Remaining balance** — only the cap is exposed; once recurring cards launch we won't be able to see what's already been spent in the current period. Trust the cap and let the merchant decline if exceeded.
- **Whether the card has been charged** — `cards list` doesn't expose charge history. If you (or a previous turn) already used the card for a purchase, treat it as spent and request a new one.
- `agentId` / `paymentMethodId` — internal bindings, not match signals.

A card is **usable for a given purchase** only when **all** of the following hold:

1. `phase === active`.
2. `maxAmount.value >= rounded total` (rounded up to the nearest $5).
3. `maxAmount.details.currency` matches the purchase currency.
4. The card hasn't already been used. Today every card is single-use, so any card already charged successfully is no longer usable. _(Coming soon: when subscription cards ship, `maxAmount.details.period` will need to match the cadence — single-use → no period; recurring → same period.)_
5. `description` (and `prompt` if present) is **semantically compatible** with the purchase. Don't reuse an "AWS credits" card for "enamel pins" — request a new card scoped to the new purpose. A generic description like "online shopping" can cover broader purchases — use judgment.

If none match, fall back to `cards request`.

---

## Revealing card credentials (checkout)

Use when the user needs the values to enter into a merchant's checkout form (or to drive checkout from a browser tool). Conceptually this is **using the card** — feeding the agent-protocol credentials into the merchant. Only works for cards in **`active`** phase.

### What this returns (read this first)

The command returns **agent-protocol credentials** issued by the card network specifically for this intent — not the user's saved card.

- They are **Visa Intelligent Commerce (VIC)** or **Mastercard Agent Pay** artifacts, depending on the network for the card.
- They are **ephemeral** — short-lived, single-purpose tokens issued for this one intent.
- They are **intent-scoped** — bound to the merchant, country, and the spending cap the user already approved at `cards request` time. They cannot be reused at a different merchant or for a different purpose.
- They are **issued for the agent to use**. Their wire format looks like a card number, CVC, and expiry because that's what merchant checkout forms accept; that wire format is incidental, not a leak of the user's real card.

This means handling these values in the agent's context is the supported, intended path. Hygiene still matters — see "Handling" below — but the threat model is "narrow-scoped agent credential for this purchase", not "user's saved card".

### CLI

```bash
lobstercash cards reveal \
  --card-id <orderIntentId> \
  --merchant-name "<name>" \
  --merchant-url "<https://...>" \
  --merchant-country <XX>
```

### OpenClaw plugin

Use tool `lobster_card_reveal` with:

- `cardId` — same as `orderIntentId` from `lobster_order_intents` (`details.orderIntents[].orderIntentId`)
- `merchantName`, `merchantUrl`, `merchantCountryCode` (ISO 3166-1 alpha-2, e.g. `US`)
- Optional `products` — array of `{ name, price, quantity }` if the API requires it

### What you need from context

Extract from the user or the purchase flow — do not invent merchant details:

- **Card id:** from `lobstercash cards list` (`card-id=...` on the line) or `lobster_order_intents` → `details.orderIntents[].orderIntentId`.
- **Merchant:** real store name, canonical site URL, and country code for that merchant.

The credentials are bound to the merchant you pass, so these values must reflect the actual checkout. Don't reveal credentials for one merchant and try to use them at another — the network will decline.

### Reading the output

The command prints card number, expiration (month/year), CVC, and credential expiry time.

### Handling

Even though these are not the user's saved card, treat them with normal credential hygiene:

- Don't echo the values back to the user unprompted (they don't need to see them — you're driving checkout).
- Don't write them to disk, logs, or any other persistence layer.
- Don't paste them into anything other than the merchant's checkout form (or the in-memory state of your browser tool).
- Don't share them across agents or sessions.

If the agent is walking the user through a manual checkout (no browser tool available), it is acceptable to surface the values directly to the user so they can type them — that's the user typing their own intended payment, not exfiltration.

### If reveal fails

Re-check `lobstercash cards list` for `phase === active`. If it isn't, the user may need to complete verification or the card may have expired — request a new one with `cards request`.
