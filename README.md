# @crossmint/lobster-cli

CLI that gives AI agents payment tools — a blockchain wallet on Solana, virtual credit cards for buying anything online, and x402 protocol support for paying APIs — all with secure guardrails and appropriate human controls. Maintained by [Crossmint](https://crossmint.com). Agent-framework agnostic.

## Install

```bash
npm install @crossmint/lobster-cli
```

## Quick start

```bash
# 1. Set up a wallet (opens consent URL for human approval)
lobstercash setup

# 2. Check balance
lobstercash crypto balance

# 3. Send USDC
lobstercash crypto send --to <address> --amount 10
```

## For agents

If you are an AI agent using this CLI, read the skill files in `skills/` before doing anything. Start with `skills/SKILL.md` — it contains the decision tree for routing to the correct reference.

**Output contract:**

- All commands produce human-readable output to stdout
- Errors go to stderr as plain text
- Exit 0 = success, 1 = failure
- Use `lobstercash agents set-active` to change the active agent

## Global options

| Flag             | Description     | Default                    |
| ---------------- | --------------- | -------------------------- |
| `--server <url>` | Server base URL | `https://www.lobster.cash` |
| `--timeout <ms>` | Request timeout | `15000`                    |

## Commands

### `setup`

Generate a local keypair and start the consent flow. Run again after approval to finalize.

```bash
lobstercash setup
```

### `status`

Check agent status: wallet setup, balances, virtual cards, and payment readiness.

```bash
lobstercash status
```

### `examples`

Show real, working examples of what agents can do with lobster.cash.

```bash
lobstercash examples
```

### `crypto balance`

Check wallet balances.

```bash
lobstercash crypto balance
```

### `crypto send`

Send tokens in a single step (create + sign + approve).

```bash
lobstercash crypto send --to <address> --amount <amount> [--token usdc] [--timeout 60000]
```

### `crypto deposit`

Generate a deposit request URL for human approval. Bundles wallet setup if needed.

```bash
lobstercash crypto deposit --amount <amount> --description "<desc>"
```

### `crypto x402 fetch`

Fetch an x402-protected URL, automatically handling payment.

```bash
lobstercash crypto x402 fetch <url> [--network mainnet-beta] [--json '<body>'] [--header "Key: Value"] [--debug]
```

### `crypto tx create`

Create a transaction without approving it.

```bash
# Transfer
lobstercash crypto tx create --type transfer --to <address> --amount <amount> [--token usdc]

# Serialized transaction
lobstercash crypto tx create --type serialized --serialized-transaction <data>

# Contract calls
lobstercash crypto tx create --type calls --calls '<json array>'
```

### `crypto tx approve`

Approve a pending transaction by signing.

```bash
# Let the CLI sign the message from tx create
lobstercash crypto tx approve --id <txId> --message <msg> [--encoding utf8]

# Or provide a pre-computed signature
lobstercash crypto tx approve --id <txId> --signature <sig>
```

### `crypto tx status`

Check transaction status. Waits for a terminal state by default.

```bash
lobstercash crypto tx status --id <txId> [--timeout 60000]
```

### `cards request`

Generate a virtual card request URL for human approval.

```bash
lobstercash cards request --amount <amount> --description "<description>"
```

### `cards list`

List virtual card order intents for the agent wallet.

```bash
lobstercash cards list
```

### `cards reveal`

Reveal virtual card credentials for a purchase.

```bash
lobstercash cards reveal --card-id <id> --merchant-name "<name>" --merchant-url "<url>" --merchant-country <XX>
```

## Agent Management

Multiple agents are managed with `lobstercash agents register`, `lobstercash agents list`, and `lobstercash agents set-active`. Each agent has its own keypair, tokens, and smart wallet. Other commands use whichever agent is currently active.

### `agents register`

Registers a new agent on the server and sets it as active locally.

```bash
lobstercash agents register --name <name> [--description <desc>] [--image-url <url>]
```

### `agents list`

Lists all local agents and shows which is active.

```bash
lobstercash agents list
```

### `agents set-active`

Sets the active agent.

```bash
lobstercash agents set-active <agentId>
```

## Library usage

The package also exports all core functions for programmatic use:

```typescript
import {
  parseConfig,
  getOrCreateWallet,
  ensureAuthenticated,
  withAuthenticatedApi,
  getWalletBalance,
  createTransaction,
  approveTransaction,
  runSetupFlow,
} from "@crossmint/lobster-cli";
```

See [src/index.ts](src/index.ts) for the full list of exports.

## Storage

Agent data and wallet keys are stored in `~/.lobster/agents.json` (mode `0600`). Override the directory with the `LOBSTER_CASH_WALLETS_DIR` environment variable. On first run, existing data is auto-migrated from `wallets.json` or the legacy path `~/.openclaw/lobster-cash/wallets.json`.

## Output

All commands produce human-readable output to stdout. Errors go to stderr as plain text.

## License

MIT
