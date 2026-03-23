# Request Deposit — Fund the Wallet with USDC

Request a USDC deposit into the agent's wallet. Generates an approval URL where the user can deposit funds. If the wallet isn't configured yet, this command bundles setup automatically.

## Command

```bash
lobstercash request deposit --amount <amount> --agent-id <id>
```

## When to use

- The user wants to add funds or top up their wallet
- Balance is insufficient for a crypto operation (`send`, `x402 fetch`, `tx`)
- The wallet isn't configured and the user needs crypto (not card) — this bundles setup + deposit in one step

## What you need before running

- `amount`: how much USDC to deposit (e.g. `25.00`)

Calculate the amount based on what the user needs. If topping up for a specific operation, use: `needed amount - current balance`.

Always check balance first with `lobstercash balance --agent-id <id>` to know the current state.

## Reading the output

The output contains:

- `agentId`: the agent this deposit is for
- `amount`: the requested deposit amount in USDC
- `approvalUrl`: the URL the user must open to deposit
- `setupSessionId`: present if wallet setup was bundled (first-time use)

## After running

Show the approval URL to the user:

> To deposit $[amount] USDC, open this link:
>
> [approvalUrl]
>
> Come back here when you've completed the deposit.

Do not proceed until the user confirms they have deposited.

## After user confirms

Run `lobstercash status --agent-id <id>` to verify the deposit landed and the wallet is ready. Then proceed with the user's original task (`send`, `x402 fetch`, etc.).

## Gotchas

- If the wallet isn't configured, setup is bundled automatically — do not run `lobstercash setup` first
- Only needed for crypto operations — virtual cards (`request card`) don't require USDC, so don't use this when `paymentMethods` includes `card`
- Always check balance first (`lobstercash balance`) — know the current state before requesting
- Write operation — do not retry automatically or if the user declines
