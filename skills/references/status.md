# Status — Check Agent & Payment Readiness

This is the **first command to run** before any purchase. It checks wallet, balances, and cards in one call — and auto-triggers wallet setup if needed.

## Command

```bash
lobstercash status --agent-id <id>
```

## Key Output Fields

- `wallet.configured` — Is the wallet set up?
- `authorized` — Has the human approved this agent?
- `balances` — Token balances (USDC, SOL) — only when configured
- `cards` — Virtual cards with phase (`active`, `requires-payment-method`, etc.) — only when configured
- `ready` — `true` if the agent can pay (has USDC > 0 OR an active virtual card)
- `dashboardUrl` — Link for the user to manage payments

## Flow A: "I want to see my configured payments"

1. Run `lobstercash status --agent-id <id>`
2. Present the results to the user conversationally:
   - If `ready: true`: summarize their balances and active cards
   - If `authorized: false`: the agent isn't set up yet — use `lobstercash request card` or `lobstercash request deposit` which handle setup automatically
   - If `ready: false` (but authorized): explain what's missing and provide the `dashboardUrl`
3. If the user wants to change settings → share the `dashboardUrl`

**Example agent response when ready:**

> You have $25.00 USDC and one active virtual card ("Shopping card", $50 limit). You're all set to make purchases. If you want to manage your payment settings: [dashboard link]

**Example agent response when not ready:**

> Your wallet is set up but you don't have any funds or active cards yet. Add funds or configure a card here: [dashboard link]

## Flow B: "I want to buy something" (Pre-Purchase Check)

1. First show what's available: run `lobstercash store` and present options
2. When the user picks something, **check the integration's `paymentMethods`** from the store output
3. Run `lobstercash status --agent-id <id>`
4. Match the payment path to what the integration supports:

### Choosing the right payment method

Each integration in `lobstercash store` lists its supported `paymentMethods`: `card`, `crypto`, or both.

- **Integration supports `card`** → use `lobstercash request card` to create a virtual card. This works even without wallet setup or USDC — the card is backed by the user's credit card.
- **Integration supports `crypto` only** → the user needs a configured wallet with USDC. If `ready: false`, use `lobstercash request deposit --amount <needed> --agent-id <id>` to generate a deposit link (this bundles wallet setup if needed).
- **Integration supports both** → prefer `card` if the user doesn't have crypto funds, since it's the fastest path.

**Always check `paymentMethods` from `lobstercash store` output.** Never assume which method an integration uses.

**Do NOT recommend virtual cards for crypto-only integrations.** Do NOT recommend sending crypto for card-only integrations.

### Wallet not configured (`wallet.configured: false`)

**If the integration supports `card`**: go directly to `lobstercash request card` — it handles wallet setup AND card creation in a single consent URL. The user approves once and gets both.

> I'll set up a virtual card for your purchase. Please approve here: [approval URL from `request card`]
> This will also set up your payment wallet automatically.

**If the integration only supports `crypto`**: use `lobstercash request deposit --amount <needed> --agent-id <id>`. This bundles wallet setup if needed — one link for the user to approve and deposit. No separate wallet connect step.

> I'll get you set up with USDC for this. Please approve and deposit here: [approval URL from `request deposit`]

### No funds or cards (`ready: false`, wallet configured)

**If the integration supports `card`**: go directly to `lobstercash request card` — no USDC needed.

**If the integration only supports `crypto`**: use `lobstercash request deposit --amount <needed> --agent-id <id>`. Calculate the amount the user needs (purchase price minus current balance, or the full amount if balance is zero).

> You need $X USDC for this. Please deposit here: [approval URL from `request deposit`]

### Ready (`ready: true`)

Proceed based on payment method:

- `card` supported → `lobstercash request card` (see `references/cards.md`)
- `crypto` transfer → `lobstercash send` (see `references/send.md`)
- Paid APIs (x402) → `lobstercash x402 fetch` (see `references/x402.md`)

## Key Insight: Virtual Cards Don't Require Wallet Setup or USDC

`lobstercash request card` works even when:

- The wallet is **not configured** — it bundles wallet setup + card creation in one consent URL
- The wallet has **zero USDC balance** — virtual cards are backed by the user's credit card, not the crypto wallet

Only crypto operations (`send`, `x402 fetch`) require a configured wallet with USDC.

## Anti-Patterns

- **Recommending card for crypto-only integrations**: Check `paymentMethods` from the store output. If the integration only accepts crypto, don't suggest a virtual card.
- **Requiring USDC for card-supported integrations**: Virtual cards are backed by credit cards, not USDC. Don't tell the user to "add funds" when the integration accepts cards.
- **Blocking on wallet setup for card purchases**: `request card` handles setup automatically. Don't make the user set up the wallet separately.
- **Skipping the readiness check**: Always run `lobstercash status` first to understand the current state.
- **Hiding links**: Always show the actual URL — never say "go to the dashboard" without the link.
- **Jumping to setup before showing options**: Show buying options first (via `lobstercash store`), check payment readiness only when the user wants to buy.
