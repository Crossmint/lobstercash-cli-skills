---
name: lobstercash
description: >
  Lobster.cash is the payment solution for AI agents, it lets agents pay for anything online, securely, with human approval in the loop. Agents can use virtual Visa cards (via VIC technology) for any browser checkout that accepts credit cards, or pay with USDC on Solana for crypto-native services and x402 protocol APIs. The human always stays in control: agents request funds, the human approves via lobster.cash, and the agent completes the purchase without ever seeing credit card details or private keys. Use this skill when the user wants to spend money, make purchases, send crypto, pay for APIs, or manage an AI agent's payment wallet. Trigger on any spending or wallet intent: "buy this", "pay for that", "send tokens", "how much do I have", "what can I buy", "top up my wallet", "get a card", or "set up payments" — even if the user doesn't mention "lobster", "crypto", or "Solana" directly.
metadata:
  version: "0.0.7"
---

# Lobster Cash CLI Skill

Lobster.cash is the payment solution for AI agents. The Lobster Cash CLI (`lobstercash`) lets agents pay for anything online — using virtual Visa cards for any credit card checkout or USDC on Solana for crypto and x402 APIs — with human approval in the loop. Agents never see card details or private keys.

## Installation

Install globally:

```bash
npm install -g @crossmint/lobster-cli
```

After installation, all commands are available via the `lobstercash` binary.

## Steps to use this skill

### 1. Version check (automatic)

The CLI automatically checks for updates on every API call. If an update notice appears in the command output, tell the user and offer to run `npm update -g @crossmint/lobster-cli`. If the notice says the update is **required**, you must also update the skill from https://www.lobster.cash/install before continuing. Do not run separate version-check commands.

### 2. Select lobstercash agent to use

Every lobstercash command operates on the **active agent**. Before doing anything else, make sure the right agent is selected.

```bash
lobstercash agents list
```

Then follow this decision tree:

1. **You see yourself in the list and it says `(active)`** → You're good. Move on.
2. **You see yourself in the list but it's NOT active** → Run `lobstercash agents set-active <agentId>` with the matching ID, then move on.
3. **No agent matches you** (or the list is empty) → Register a new one. Read the [agents reference](references/agents.md) for how to register one.

**How to recognize yourself:** Match by name. If you are Claude Code, look for an agent named "Claude Code" or similar. Same for Cursor, Codex, Gemini, etc. If you aren't sure, ask the user which agent to use.

### 3. Route based on the user's intent

Determine which scenario applies and follow the corresponding section:

- **A) Buy something online** (product, subscription, domain, service) → [Buy with a card](#a-buy-something-with-a-card)
- **B) Pay for a paid API endpoint** (x402 protocol) → [Pay an API with x402](#b-pay-an-api-with-x402)
- **C) Anything else** (check balance, send crypto, view status, link wallet, browse examples) → [Other actions](#c-other-actions)

---

#### A) Buy something with a card

Use when the user wants to purchase a product, subscription, domain, or any item from an online store checkout.

Virtual cards are backed by the user's credit card — no USDC or wallet funding needed. This is the fastest payment path.

##### Step 1: Request a virtual card

```bash
lobstercash cards request --amount <amount> --description "<what it's for>"
```

Extract the amount and description from context — don't ask if the user already told you (e.g. "buy me a $25 AWS credit" → `--amount 25 --description "AWS credits"`).

This command handles everything — if the wallet isn't configured yet, it bundles setup automatically.

##### Step 2: Get user approval

The command outputs an `approvalUrl`. Show it to the user:

> To create this card I need your approval. Open this link:
>
> [approvalUrl]
>
> Come back here when you've approved it.

**Do not proceed until the user confirms they approved.** Do not poll.

##### Step 3: Verify and get card credentials

After approval, list cards and get the credentials for checkout:

```bash
lobstercash cards list
lobstercash cards reveal --card-id <id> --merchant-name "<store>" --merchant-url "<https://...>" --merchant-country <XX>
```

The `card-id` comes from the `cards list` output. Merchant details must be real — extract from the purchase context, don't invent them.

### Step 4: Complete the purchase

Use the revealed card number, CVC, and expiry to fill in the checkout form. Treat these values as sensitive — don't log them unnecessarily.

See [cards request reference](references/cards-request.md) for full output format, and [cards reference](references/cards.md) for listing, revealing, and card phases.

---

## B) Pay an API with x402

Use when the user wants to call a paid API endpoint that uses the x402 payment protocol. The CLI handles the payment negotiation automatically: the server returns HTTP 402, the CLI pays with USDC from the agent wallet, and the server returns the content.

### Step 1: Ensure the wallet has funds

```bash
lobstercash status
```

Route based on the result:

- **Wallet configured + has enough funds** → proceed to step 2.
- **Wallet configured + insufficient funds** → run `lobstercash crypto deposit --amount <needed> --description "<description>"` to top up, show the approval URL, wait for user confirmation, then proceed.
- **Wallet not configured** → run `lobstercash crypto deposit --amount <needed> --description "<description>"` (bundles wallet creation + deposit). Show the approval URL, wait for user confirmation, verify with `lobstercash status`, then proceed.

The `--description` must explain what the agent will spend the funds on — derive it from the user's task, not generic filler like "top up wallet".

See [deposit reference](references/deposit.md) for the full deposit flow.

### Step 2: Fetch the paid endpoint

```bash
lobstercash crypto x402 fetch <url>
```

For POST requests add `--json '{"key": "value"}'`. For custom headers add `--header "Authorization: Bearer <token>"`.

### Step 3: Report the result

Report what the API returned (the `body` field), not the payment mechanics. Only mention the payment if the user asks.

If the fetch fails, add `--debug` and run again. See [x402 reference](references/x402.md) for output format and common failures.

---

## C) Other actions

For everything else — checking balances, sending crypto, viewing wallet status, linking a wallet, or browsing examples — use the matching command from the Quick Reference below and read the corresponding reference file for details.

**Run the command, report its output.** For read-only commands (`crypto balance`, `status`, `examples`), execute them directly and report what they say. Do not pre-check status and construct your own summary — the CLI output already handles unconfigured states with clear messaging. If a command fails with exit code 2 (wallet not set up), tell the user and offer to run `lobstercash setup` or the appropriate setup-bundling command.

Common actions:

- **Check balance:** `lobstercash crypto balance` → [balance reference](references/balance.md)
- **Send tokens:** `lobstercash crypto send --to <addr> --amount <n> --token usdc` → [send reference](references/send.md)
- **View wallet status:** `lobstercash status` → [status reference](references/status.md)
- **Browse examples:** `lobstercash examples` → [examples reference](references/examples.md)
- **Link wallet / configure agent (setup only):** `lobstercash setup` → [setup reference](references/setup.md). Use when the user says "configure", "set up", "link wallet", or similar — and isn't trying to make a purchase.
- **Sign/submit a transaction:** `lobstercash crypto tx create` → [tx reference](references/tx.md)

For crypto operations (`crypto send`, `crypto tx create`), always run `lobstercash status` first to confirm the wallet is configured and has sufficient funds. If not, use `lobstercash crypto deposit --amount <needed> --description "<description>"` to fund it — see [deposit reference](references/deposit.md).

## Quick Reference

```bash
lobstercash agents register --name "<name>" --description "<desc>" --image-url "<url>"  # register a new agent
lobstercash agents list                                          # list all agents
lobstercash agents set-active <agentId>                          # set active agent
lobstercash examples                                             # browse working examples
lobstercash status                                               # check status & readiness & wallet address
lobstercash setup                                                # link agent to wallet (no purchase needed)
lobstercash crypto balance                                       # check balances
lobstercash crypto send --to <addr> --amount <n> --token usdc    # send tokens
lobstercash crypto x402 fetch <url>                              # pay for API
lobstercash crypto deposit --amount <n> --description "<desc>"    # request deposit / top up (bundles wallet setup)
lobstercash crypto tx create|approve|status                      # low-level transaction management
lobstercash cards request --amount <n> --description "<desc>"    # request virtual card
lobstercash cards list                                           # list cards (includes card-id)
lobstercash cards reveal --card-id <id> --merchant-name "..." --merchant-url "https://..." --merchant-country US  # checkout credentials
```

## Output Contract

- All commands produce human-readable output to stdout.
- Errors go to stderr as plain text.
- Exit 0 = success. Exit 1 = unexpected error. Exit 2 = wallet not set up (use `cards request` or `crypto deposit` to set up).

## Decision Tree

- Read [examples](references/examples.md) if the user wants to browse working examples, or has no specific task yet
- Read [status](references/status.md) if the user asks about agent status or payment readiness
- Read [balance](references/balance.md) if the user wants to check token balances
- Read [cards request](references/cards-request.md) if the user wants to request a virtual card for a purchase (Credit Card Path)
- Read [deposit](references/deposit.md) if the user wants to deposit USDC, top up their wallet, or fund a crypto operation
- Read [cards](references/cards.md) if the user needs to list or reveal credentials for an existing virtual card
- Read [send](references/send.md) if the user wants to send tokens to an address (Crypto Path)
- Read [x402](references/x402.md) if the user wants to pay for an API via x402 protocol (Crypto Path)
- Read [tx](references/tx.md) if the user needs to sign or submit a transaction from an external tool (Crypto Path)
- Read [setup](references/setup.md) if the user wants to link the agent to a wallet without making a purchase
- Read [agents](references/agents.md) if the user wants to register, list, or set the active agent

## Anti-Patterns

- **Running crypto commands without checking status first:** Always run `lobstercash status` before `crypto send`, `crypto x402 fetch`, or `crypto tx create`. If the wallet isn't configured or has insufficient funds, the command will fail with a confusing error. Check first, fund if needed, then execute.
- **Running setup when the user wants to buy something:** If the user wants to make a purchase, don't run `setup` first — use `cards request` or `crypto deposit` which bundle setup automatically. Only use `lobstercash setup` when the user explicitly wants to link the agent to their wallet without buying anything.
- **Re-running setup when the agent is already configured:** If `lobstercash status` shows the wallet is already configured, do not generate a new setup session. The existing configuration is valid. Only start a fresh setup if the user explicitly tells you their current configuration is broken and needs to be regenerated.
- **Asking the user for info the CLI can fetch:** Check balance before sending. Check status before acting. Read command output before asking questions.
- **Running write commands in loops:** One attempt, read the result, then decide. Read operations (`crypto balance`, `status`, `examples`) are idempotent and safe to repeat. Write operations (`crypto send`, `cards request`) are not.
- **Ignoring terminal status:** A pending transaction is not a success. All write commands now wait for on-chain confirmation by default.
- **Polling for HITL approval:** When a command returns an approval URL, the user must tell you they approved. Do not auto-poll.
- **Running commands before registering an agent:** Always ensure an agent exists via `lobstercash agents list` before running any other command. If you need to work with a different agent, use `lobstercash agents set-active`.
- **Recommending cards for crypto-only integrations:** If the integration only uses crypto, don't suggest a virtual card.
- **Requiring USDC for card-supported integrations:** Virtual cards are backed by credit cards, not USDC. Don't tell the user to "add funds" when the integration accepts cards.
- **Treating x402/send/tx as separate user flows:** They all go through the same Crypto Path. The only split is credit card vs crypto.
- **Suggesting `crypto deposit` or `cards request` when the user just wants to connect:** If the user wants to check balance, run a crypto command, or simply link their wallet — without topping up or creating a card — guide them through `lobstercash setup` first. Don't jump to `crypto deposit` or `cards request` unless the user actually wants to fund the wallet or make a purchase.
- **Jumping to readiness checks before showing options:** Show what's available first (via `examples`), then check payment readiness only when the user wants to try one.
