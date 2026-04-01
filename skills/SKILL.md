---
name: lobstercash
description: >
  Use this skill when the user wants to spend money, make purchases, send crypto, pay for APIs, or manage an AI agent's payment wallet. Covers buying products online with credit cards, sending tokens, paying for x402 protocol APIs, checking balances, depositing funds, browsing available services, and signing on-chain transactions — all with secure guardrails, and appropriate human controls. Trigger on any spending or wallet intent: "buy this", "pay for that", "send tokens", "how much do I have", "what can I buy", "top up my wallet", "get a card", or "set up payments" — even if the user doesn't mention "lobster", "crypto", or "Solana" directly.
metadata:
  version: "0.0.6"
---

# Lobster Cash CLI Skill

The Lobster Cash CLI (`lobstercash`) gives AI agents payment tools — a blockchain wallet, virtual cards for buying anything online, and x402 protocol support for paying APIs — all with human approval in the loop. Use `lobstercash <command> --help` for full flag details on any command.

## Installation

Install and run the CLI via npx — no global install required:

```bash
npx @crossmint/lobster-cli --help
```

Or install globally:

```bash
npm install -g @crossmint/lobster-cli
```

After installation, all commands are available via the `lobstercash` binary.

## Preflight: Stay Current

**Run this check once at the start of each session, before executing any other `lobstercash` command.** This keeps everything up to date — similar to how zsh prompts "a new version is available" on launch.

Run both of these commands silently:

```bash
npm view @crossmint/lobster-cli version        # latest CLI version on npm
lobstercash --version                           # local CLI version (may fail if not installed)
curl -sL https://raw.githubusercontent.com/Crossmint/lobstercash-cli-skills/main/skills/SKILL.md | head -10
# compare remote metadata.version against the local one in this file's frontmatter
```

If **everything is current** (CLI version matches npm and skill `metadata.version` matches remote), proceed silently — do not mention versioning to the user.

If **anything is outdated**, tell the user in a single message before doing anything else. Include only the items that need updating:

> "New lobster.cash updates are available:
> - CLI: vX.Y.Z → vA.B.C
> - Skill instructions: vX.Y.Z → vA.B.C
>
> Want me to update before we continue?"

If the user says yes, run the applicable updates:

- **CLI update (global install):** `npm update -g @crossmint/lobster-cli`
- **CLI update (npx users):** `npx @crossmint/lobster-cli@<version> --help` (use the exact version from `npm view` above to bust the npx cache)
- **Skill update (OpenClaw agents):** `openclaw skill update lobstercash`
- **Skill update (other agents):**
  ```bash
  npx skills add https://github.com/Crossmint/lobstercash-cli-skills --global --yes
  ```

If the skill files were updated, **re-read this SKILL.md from the top** — instructions or references may have changed.

If the user declines, continue with the current versions — do not block.

**Before any crypto command** (`send`, `x402 fetch`, `tx create`), always run `lobstercash status` first. If the wallet isn't configured or funds are insufficient, route through `request deposit` — it bundles wallet setup automatically. For card purchases, `request card` also bundles setup. Only use `lobstercash setup` when the user explicitly wants to link the agent to their wallet without buying anything.

## Getting Started

If the user already knows what they want to buy, skip straight to "Buy Something" below.

If the user doesn't have a specific purchase intent (e.g. "what can I do with lobster.cash?", "show me what's available"), run discovery first:

```bash
lobstercash store
```

Present 2-3 interesting options from the results conversationally. Don't dump the full list — pick items relevant to what the user has been doing or asking about.

Once the user picks something (or if they came in already knowing what they want), route to one of the branches below.

## Step 0: Ensure an Agent Exists

```bash
lobstercash agents list
```

If no agents exist, register one with a unique, descriptive name and description. Always include `--image-url` if you have an avatar or logo. Do not use generic names like "My Agent".

```bash
lobstercash agents register --name "<descriptive name>" --description "<what the agent does>" --image-url "<avatar url>"
```

This registers the agent on the server and sets it as active.

## Branch 1: View Configured Payments

**Trigger:** User asks about their payment methods, wallet status, balances, or configured payments ("show me my payments", "what's my balance", "am I set up?").

```bash
lobstercash status
```

**What to tell the user:**

- Payment methods with their balances
- Active virtual cards (if any)
- Share the `dashboardUrl` so they can manage settings themselves: https://www.lobster.cash/dashboard
- If the wallet isn't authorized or configured, just report the current state. Don't explain what's missing or prompt the user to fix it — configuration is lazy. When the user later says they want to buy something or add funds, the right setup path (crypto or card) kicks in automatically as part of Branch 2.

See [status reference](references/status.md) for the full output format and examples.

## Branch 2: Buy Something

**Trigger:** User wants to purchase an item, make a payment, send crypto, call a paid API, or execute any transaction. This is the single branch for **all actions that involve spending money**.

### Step 1: Run `lobstercash store` and check the payment method

**Always run `lobstercash store` first.** Do not guess which payment method an integration uses — check the `paymentMethods` field in the store output. Then route:

- **`card`** → Credit Card Path (Step 2a)
- **`crypto`** → Crypto Path (Step 2b)
- **Both** → Prefer card if the user has no crypto funds (fastest path)

**Critical:** x402 payments, USDC transfers, send operations, transaction signing — they are ALL crypto. Everything that isn't a credit card goes through the Crypto Path. Do not treat them as separate flows.

### Step 2a: Credit Card Path

**When:** The integration's `paymentMethods` includes `card` (check the store output — do not assume).

Virtual cards do NOT require wallet setup or USDC — they're backed by the user's credit card. This is the fastest path. Don't tell the user to "add funds" when the integration accepts cards.

```bash
lobstercash request card --amount <amount> --description "<what it's for>"
```

This command handles everything — if the wallet isn't configured yet, it bundles setup automatically. The user approves once and gets both wallet setup + card creation.

Show the approval URL to the user, wait for them to confirm they approved, then list and reveal card credentials as needed. See [request card reference](references/request-card.md) for details and [cards reference](references/cards.md) for listing/revealing credentials.

### Step 2b: Crypto Path

**When:** The integration's `paymentMethods` is `crypto` only (check the store output), OR the user wants to send tokens, pay an x402 API, or execute any on-chain transaction.

First, check wallet status and balance:

```bash
lobstercash status
```

Then route based on the result:

- **Wallet configured + has enough funds** → Proceed with the operation (`send`, `x402 fetch`, `tx create`, etc.)
- **Wallet configured + insufficient funds** → Run `lobstercash request deposit --amount <needed>` to top up, then proceed
- **Wallet not configured** → Run `lobstercash request deposit --amount <needed>` — this bundles wallet creation + deposit in one step. Show the approval URL, wait for user to confirm, then verify with `lobstercash status` and proceed.

The specific CLI command used after funds are confirmed depends on what the user is doing, but the path to get there is always the same: check wallet → check balance → ensure funds → execute.

See [request deposit reference](references/request-deposit.md) for the deposit flow, and [send](references/send.md), [x402](references/x402.md), [tx](references/tx.md) for each operation after funding.

## Branch 3: Link Agent to Wallet (Setup Only)

**Trigger:** User wants to connect the agent to their wallet without making a purchase ("link my wallet", "set up the agent", "authorize this agent", "connect my wallet").

**First, check if the agent is already configured:**

```bash
lobstercash status
```

If the wallet is already configured, tell the user the agent is already set up and show the existing wallet info. **Do not start a new setup session.** Only proceed with a fresh setup if the user explicitly says their current configuration is broken or they need to generate a new one.

**If not yet configured:**

```bash
lobstercash setup
```

This creates a local keypair, starts a setup session, and returns a consent URL. Show the URL to the user and wait for them to approve — do not poll.

After the user confirms they approved, run `lobstercash setup` again to finalize. The CLI checks the session status automatically.

Once configured, the agent is ready for any future operation (crypto or card). If the user then wants to buy something, route to Branch 2.

See [setup reference](references/setup.md) for output formats, status codes, and edge cases.

## Quick Reference

```bash
lobstercash agents register --name "<name>" --description "<desc>" --image-url "<url>"  # register a new agent
lobstercash agents list                                          # list all agents
lobstercash agents set-active <agentId>                           # set active agent
lobstercash store                                                # browse integrations
lobstercash status                                               # check status & readiness
lobstercash setup                                                # link agent to wallet (no purchase needed)
lobstercash balance                                              # check balances
lobstercash request card --amount <n> --description "<desc>"     # request virtual card
lobstercash cards list                                           # list cards (includes card-id)
lobstercash cards reveal --card-id <id> --merchant-name "..." --merchant-url "https://..." --merchant-country US  # checkout credentials
lobstercash request deposit --amount <n>                         # request deposit / top up (bundles wallet setup)
lobstercash send --to <addr> --amount <n> --token usdc           # send tokens
lobstercash x402 fetch <url>                                     # pay for API
```

## Output Contract

- All commands produce human-readable output to stdout.
- Errors go to stderr as plain text.
- Exit 0 = success. Exit 1 = unexpected error. Exit 2 = wallet not set up (use `request card` or `request deposit` to set up).

## Decision Tree

- Read [store](references/store.md) if the user wants to browse available services, integrations, or has no specific task yet
- Read [status](references/status.md) if the user asks about agent status or payment readiness
- Read [balance](references/balance.md) if the user wants to check token balances
- Read [request card](references/request-card.md) if the user wants to request a virtual card for a purchase (Credit Card Path)
- Read [request deposit](references/request-deposit.md) if the user wants to deposit USDC, top up their wallet, or fund a crypto operation
- Read [cards](references/cards.md) if the user needs to list or reveal credentials for an existing virtual card
- Read [send](references/send.md) if the user wants to send tokens to an address (Crypto Path)
- Read [x402](references/x402.md) if the user wants to pay for an API via x402 protocol (Crypto Path)
- Read [tx](references/tx.md) if the user needs to sign or submit a transaction from an external tool (Crypto Path)
- Read [setup](references/setup.md) if the user wants to link the agent to a wallet without making a purchase
- Read [agents](references/agents.md) if the user wants to register, list, or set the active agent

## Anti-Patterns

- **Running crypto commands without checking status first:** Always run `lobstercash status` before `send`, `x402 fetch`, or `tx create`. If the wallet isn't configured or has insufficient funds, the command will fail with a confusing error. Check first, fund if needed, then execute.
- **Running setup when the user wants to buy something:** If the user wants to make a purchase, don't run `setup` first — use `request card` or `request deposit` which bundle setup automatically. Only use `lobstercash setup` when the user explicitly wants to link the agent to their wallet without buying anything.
- **Re-running setup when the agent is already configured:** If `lobstercash status` shows the wallet is already configured, do not generate a new setup session. The existing configuration is valid. Only start a fresh setup if the user explicitly tells you their current configuration is broken and needs to be regenerated.
- **Asking the user for info the CLI can fetch:** Check balance before sending. Check status before acting. Read command output before asking questions.
- **Running write commands in loops:** One attempt, read the result, then decide. Read operations (`balance`, `status`, `store`) are idempotent and safe to repeat. Write operations (`send`, `request`) are not.
- **Ignoring terminal status:** A pending transaction is not a success. All write commands now wait for on-chain confirmation by default.
- **Polling for HITL approval:** When a command returns an approval URL, the user must tell you they approved. Do not auto-poll.
- **Running commands before registering an agent:** Always ensure an agent exists via `lobstercash agents list` before running any other command. If you need to work with a different agent, use `lobstercash agents set-active`.
- **Recommending cards for crypto-only integrations:** Check `paymentMethods` from the store output. If the integration only accepts crypto, don't suggest a virtual card.
- **Requiring USDC for card-supported integrations:** Virtual cards are backed by credit cards, not USDC. Don't tell the user to "add funds" when the integration accepts cards.
- **Treating x402/send/tx as separate user flows:** They all go through the same Crypto Path. The only split is credit card vs crypto.
- **Jumping to readiness checks before showing options:** Show what's available first (via `store`), then check payment readiness only when the user wants to buy.
- **Assuming an integration's payment method:** Never guess whether an integration uses `card` or `crypto`. Always run `lobstercash store` and read the `paymentMethods` field before choosing a path.
