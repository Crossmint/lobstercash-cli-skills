# Permissions: Card List — List Existing Virtual Cards

List virtual cards on the agent wallet, including their status, spending caps, and `card-id` values. This command shows card metadata only — **not** payment credentials. To get the credentials needed for checkout, use [card-use.md](card-use.md).

## Command

```bash
lobstercash permissions card list
```

## Reading the output

One line per card:

```
  <description>  $<amount> <currency> limit  [<phase>]  card-id=<orderIntentId>
```

Use `card-id` as the `<id>` argument when running `lobstercash card use <id>`. In OpenClaw, the same value is `orderIntentId` on each item in `lobster_order_intents` → `details.orderIntents`.

Possible phase values:

- `active` — card is ready to use; credentials can be retrieved for checkout
- `requires-payment-method` — no valid card on file; the user must add or update their payment method at lobster.cash/dashboard before this card can activate
- `requires-verification` — the user's bank requires additional authentication (e.g. 3D Secure); they must complete verification in the browser at the approval link
- `expired` — card is no longer valid; the user needs to request a new one

## When to use

- After the user approves a card request — to confirm the card is `active` and grab its `card-id`.
- When the user asks "do I have any cards".
- To check the status of a specific card.
- **Before creating a new card for a purchase** — always list first and check whether an existing card can be reused (see "Reusing an existing card" below).

## Reporting to the user

List only `active` cards unless the user asks for all.

> You have [n] active card(s): [description] with a $[amount] limit.

## Reusing an existing card

When you're about to create a new card for a purchase, run `permissions card list` first and check whether one already covers the purchase. Reusing a card avoids spamming the user with another approval link.

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
- **Whether the card has been charged** — `permissions card list` doesn't expose charge history. If you (or a previous turn) already used the card for a purchase, treat it as spent and request a new one.
- `agentId` / `paymentMethodId` — internal bindings, not match signals.

A card is **usable for a given purchase** only when **all** of the following hold:

1. `phase === active`.
2. `maxAmount.value >= rounded total` (rounded up to the nearest $5).
3. `maxAmount.details.currency` matches the purchase currency.
4. The card hasn't already been used. Today every card is single-use, so any card already charged successfully is no longer usable. _(Coming soon: when subscription cards ship, `maxAmount.details.period` will need to match the cadence — single-use → no period; recurring → same period.)_
5. `description` (and `prompt` if present) is **semantically compatible** with the purchase. Don't reuse an "AWS credits" card for "enamel pins" — request a new card scoped to the new purpose. A generic description like "online shopping" can cover broader purchases — use judgment.

If none match, fall back to [`permissions card request`](permissions-card-request.md).
