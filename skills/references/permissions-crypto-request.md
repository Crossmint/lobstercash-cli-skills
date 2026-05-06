# Permissions: Crypto Request — Fund the Wallet with USDC

Request USDC funding for the agent's wallet. Generates an approval URL where the user can deposit funds. If the wallet isn't linked yet, this command bundles wallet linking automatically — it is the only path that links the wallet, since `lobstercash setup` no longer exists.

## Command

```bash
lobstercash permissions crypto request --amount <amount> --description "<desc>"
```

## When to use

- The user wants to add funds or top up their wallet.
- Balance is insufficient for a crypto operation (`crypto send`, `crypto x402 fetch`, `crypto tx`).
- The wallet isn't linked yet and the user wants to do anything with crypto — this bundles wallet linking + funding.
- The user wants to "link my wallet" or "set up payments" without a specific purchase in mind. See [Linking the wallet without a specific purchase](#linking-the-wallet-without-a-specific-purchase) below.

## What you need before running

- `amount`: how much USDC to request (e.g. `25.00`). Calculate based on what the user needs. If topping up for a specific operation: `needed amount - current balance`.
- `description`: what the agent will spend the funds on — derived from the user's task, not generic filler. Good: `"Pay for 3 Exa searches on competitor pricing"`. Bad: `"Top up wallet"`, `"Fund wallet for API calls"`.

Always check balance first with `lobstercash crypto balance` to know the current state.

## Reading the output

The output contains:

- `agentId`: the agent this request is for
- `amount`: the requested funding amount in USDC
- `description`: what the funds are for
- `approvalUrl`: the URL the user must open to approve
- `setupSessionId`: present if wallet linking was bundled (first-time use)

## After running

Show the approval URL to the user:

> To fund $[amount] USDC, open this link:
>
> [approvalUrl]
>
> Come back here when you've completed the funding.

Do not proceed until the user confirms they have funded the wallet. Do not poll.

## After user confirms

Run `lobstercash status` to verify the funds landed and the wallet is ready. Then proceed with the user's original task (`crypto send`, `crypto x402 fetch`, etc.).

## Linking the wallet without a specific purchase

There is no standalone "link wallet" command — wallet linking is always bundled into `permissions crypto request` (or `permissions card request` if the user is paying with a card). When the user just wants to connect their wallet without buying anything or topping up:

1. Ask them what they intend to do with crypto next (top up for an API call, send tokens, etc.) and pick a small amount that fits that purpose.
2. Use that purpose as the `--description` and that amount as `--amount`.
3. Run `permissions crypto request` and walk them through the approval URL.

If they truly have no purpose yet and just want to connect, ask them once. If they confirm they just want to link, use a small placeholder amount (e.g. `1.00`) and a description like `"Initial wallet linking"` so the approval URL still has meaningful intent. Avoid creating a top-up larger than they actually need.

## Gotchas

- Linking is bundled — there is no `lobstercash setup` to run separately.
- Only needed for crypto operations — virtual cards (`permissions card request`) don't require USDC, so don't use this path when `paymentMethods` includes `card`.
- Always check balance first (`lobstercash crypto balance`) — know the current state before requesting.
- Write operation — do not retry automatically and do not retry if the user declines.
