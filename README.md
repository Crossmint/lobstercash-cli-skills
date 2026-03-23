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
lobstercash balance

# 3. Send USDC
lobstercash send --to <address> --amount 10
```

## For agents

If you are an AI agent using this CLI, read the skill files in `skills/` before doing anything. Start with `skills/SKILL.md` — it contains the decision tree for routing to the correct reference.

**Output contract:**

- All commands produce human-readable output to stdout
- Errors go to stderr as plain text
- Exit 0 = success, 1 = failure
- Most commands accept `--agent-id <id>` to target a specific agent's wallet

## Global options

| Flag             | Description     | Default                    |
| ---------------- | --------------- | -------------------------- |
| `--server <url>` | Server base URL | `https://www.lobster.cash` |
| `--timeout <ms>` | Request timeout | `15000`                    |

## Commands

### `setup`

Generate a local keypair and start the consent flow. Run again after approval to finalize.

```bash
lobstercash setup [--agent-id <id>]
```

### `balance`

Check wallet balances.

```bash
lobstercash balance [--agent-id <id>]
```

### `send`

Send tokens in a single step (create + sign + approve).

```bash
lobstercash send --to <address> --amount <amount> [--token usdc] [--timeout 60000] [--agent-id <id>]
```

### `tx create`

Create a transaction without approving it.

```bash
# Transfer
lobstercash tx create --type transfer --to <address> --amount <amount> [--token usdc]

# Serialized transaction
lobstercash tx create --type serialized --serialized-transaction <data>

# Contract calls
lobstercash tx create --type calls --calls '<json array>'
```

### `tx approve`

Approve a pending transaction by signing.

```bash
# Let the CLI sign the message from tx create
lobstercash tx approve --id <txId> --message <msg> [--encoding utf8]

# Or provide a pre-computed signature
lobstercash tx approve --id <txId> --signature <sig>
```

### `tx status`

Check transaction status. Waits for a terminal state by default.

```bash
lobstercash tx status --id <txId> [--timeout 60000]
```

### `request card`

Generate a virtual card request URL for human approval.

```bash
lobstercash request card --amount <amount> --description "<description>" [--agent-id <id>]
```

### `cards list`

List virtual card order intents for the agent wallet.

```bash
lobstercash cards list [--agent-id <id>]
```

### `x402 fetch`

Fetch an x402-protected URL, automatically handling payment.

```bash
lobstercash x402 fetch <url> [--network mainnet-beta] [--json '<body>'] [--header "Key: Value"] [--debug] [--agent-id <id>]
```

## Multi-agent support

Most commands accept `--agent-id <id>` to operate on a specific agent's wallet. The default agent ID is a stable device UUID persisted at `~/.lobster/device.json` (overridable via `LOBSTER_DEVICE_ID`). Each agent has its own keypair, tokens, and smart wallet.

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

Wallets are stored at `~/.lobster/wallets.json` (mode `0600`). Override the directory with the `LOBSTER_CASH_WALLETS_DIR` environment variable. On first run, existing wallets are auto-migrated from the legacy path `~/.openclaw/lobster-cash/wallets.json`.

## Output

All commands produce human-readable output to stdout. Errors go to stderr as plain text.

## License

MIT
