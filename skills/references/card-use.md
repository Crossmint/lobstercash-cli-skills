# Card Use — Get Agent-Protocol Checkout Credentials

Use this command when you need the values to enter into a merchant's checkout form (or to drive checkout from a browser tool). Only works for cards in `active` phase.

## What this returns (read this first)

The command returns **agent-protocol credentials** issued by the card network specifically for this intent — not the user's saved card.

- They are **Visa Intelligent Commerce (VIC)** or **Mastercard Agent Pay** artifacts, depending on the network for the card.
- They are **ephemeral** — short-lived, single-purpose tokens issued for this one intent.
- They are **intent-scoped** — bound to the merchant, country, and the spending cap the user already approved at `permissions card request` time. They cannot be reused at a different merchant or for a different purpose.
- They are **issued for the agent to use**. Their wire format looks like a card number, CVC, and expiry because that's what merchant checkout forms accept; that wire format is incidental, not a leak of the user's real card.

This means handling these values in the agent's context is the supported, intended path. Hygiene still matters — see "Handling" below — but the threat model is "narrow-scoped agent credential for this purchase", not "user's saved card".

## Command

```bash
lobstercash card use <id> \
  --merchant-name "<merchant name>" \
  --merchant-url "<https://merchant.com>" \
  --merchant-country <XX>
```

`<id>` is the `card-id` from `lobstercash permissions card list` (same value as `orderIntentId` in the API).

## What you need from context

Extract from the user or the purchase flow — do not invent merchant details:

- **Card id:** from `lobstercash permissions card list` (`card-id=...` on each line) or `lobster_order_intents` → `details.orderIntents[].orderIntentId`.
- **Merchant name** — the real store name (as the user would recognize it).
- **Merchant URL** — the canonical site URL (e.g. `https://amazon.com`).
- **Merchant country** — ISO 3166-1 alpha-2 (e.g. `US`, `GB`, `DE`).

The credentials are bound to the merchant you pass, so these values must reflect the actual checkout. Don't issue credentials for one merchant and try to use them at another — the network will decline and you'll have to request a new card.

## Reading the output

The command prints the card number, expiry month/year, CVC, and a credential expiry timestamp. Use them at the merchant's checkout form, then move on.

## Handling

Even though these are not the user's saved card, treat them with normal credential hygiene:

- Don't echo the values to the user unprompted (they don't need to see them — you're driving checkout).
- Don't write them to disk, logs, or any other persistence layer.
- Don't paste them into anything other than the merchant's checkout form (or the in-memory state of your browser tool).
- Don't share them across agents or sessions.

If the agent is walking the user through a manual checkout (no browser tool available), it is acceptable to surface the values directly to the user so they can type them — that's the user typing their own intended payment, not exfiltration.

## OpenClaw plugin

Use tool `lobster_card_reveal` with:

- `cardId` — same as `orderIntentId` from `lobster_order_intents` (`details.orderIntents[].orderIntentId`).
- `merchantName`, `merchantUrl`, `merchantCountryCode`.
- Optional `products` — array of `{ name, price, quantity }` if the API requires it.

## If it fails

If the command errors, re-run `lobstercash permissions card list` and confirm `phase === active`. If it isn't, the user may need to complete verification or the card may have expired — request a new one with [`permissions card request`](permissions-card-request.md).
