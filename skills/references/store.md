# Store

Browse available integrations and services the agent can use on behalf of the user. Use this to show the user what's possible when they have no specific task in mind.

## When to use

- After setup completes and the user has no specific request.
- When the user asks "what can you do?" or "what can I buy?".
- Only show once per session — do not repeat if the user already saw it.

## Command

```
lobstercash store
```

Options:

- `--category <cat>` — filter by category: shopping, ai, defi, payments, marketplace, infrastructure

## Reading the output

Each item includes:

- `name` — integration name
- `summary` — one-line description
- `category` — grouping (shopping, ai, defi, payments, marketplace, infrastructure)
- `capabilities` — list of what it can do
- `paymentMethods` — how it gets paid: `crypto` (USDC/SOL via wallet) or `card` (virtual card)
- `url` — link to the integration's documentation or skill file

## How to present

Do not dump the raw output. Summarize conversationally:

"Here's what I can help you with:

- **Shop online** — search and buy products (Purch)
- **Use AI models** — image generation, LLMs, pay per request (BlockRun)
- **Trade tokens** — swap, lend, DCA on Solana (Jupiter)
- **Launch a token** — create tokens for free, earn trading fees (ClawPump)
- **Pay for APIs** — access x402 services, even on credit (ClawCredit)

What would you like to do?"

Adapt the list based on what seems most relevant. You do not need to
list every single integration — highlight 3-5 that are most useful.

## After the user picks

Once the user picks an integration, check their balance (`lobstercash balance`)
and proceed with the relevant skill. If they need funds, guide them to
the deposit flow.
