# Cards Request — Virtual Card for Purchases

Request a virtual card backed by the user's credit card. Conceptually this is a **card permission**: the user grants the agent a scoped, capped right to charge a specific amount for a specific purpose. The actual CLI command is `lobstercash cards request`. If the wallet isn't linked yet, this command bundles wallet linking automatically.

## Command

```bash
lobstercash cards request --amount <amount> --description "<description>"
```

## What you need before running

Extract from context — do not ask if already clear:

- `amount`: maximum USD that can be spent on this card (e.g. `25.00`)
- `description`: what the card will be used for (e.g. `"AWS credits"`)

If the user said "I need a card for $25 for AWS" you already have both.

> **Note:** Subscription / recurring cards are **coming soon**. For now every card is single-use — request a fresh one per purchase. The `--period <weekly|monthly|yearly>` flag still exists in the CLI but should be omitted until subscriptions ship.

## Reading the output

The output contains:

- `agentId`: the agent this card is for
- `amount`: the requested amount
- `description`: what the card is for
- `approvalUrl`: the URL the user must open to approve
- `setupSessionId`: present if wallet linking was bundled (first-time use)

## After running

Show the approval URL to the user:

> To create this card I need your approval. Open this link:
>
> [approvalUrl]
>
> Come back here when you've approved it.

Do not proceed until the user confirms they have approved. Do not poll.

## After user approves

Run `lobstercash cards list` to verify the card was created. Find the card with matching description (`card-id=...`) and report:

> Your card is ready.

Then proceed with the user's task — see [cards.md](cards.md) for listing details, reuse rules, and getting the credentials at checkout.

## Gotchas

- Virtual cards do **not** require USDC — they're backed by the user's credit card.
- If the wallet isn't linked yet, linking is bundled automatically — do **not** run `lobstercash setup` first.
- Only use this when the integration supports `card` payments — do not recommend for crypto-only integrations.
- Write operation — do not retry automatically and do not retry if the user declines.
