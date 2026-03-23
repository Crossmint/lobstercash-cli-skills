# Setup — Link Agent to Wallet

Link this agent to your lobster.cash wallet. It gives the agent access to operate a blockchain wallet, as well as to request virtual cards and top ups. Use this when the user wants to connect the agent to their wallet **without** making a purchase. If the user wants to buy something, use `request card` or `request deposit` instead — they bundle setup automatically.

## Command

```bash
lobstercash setup --agent-id <id>
```

## Check first

Run `lobstercash status --agent-id <id>` and read the output:

- `wallet.configured: true` — wallet is ready, do not run setup.
- `wallet.configured: false` — wallet needs setup. Proceed to Step 1.

## Step 1 — Start setup

```bash
lobstercash setup --agent-id <id>
```

Parse the output:

- `status`: one of `pending_configuration`, `already_configured`, `configured`, `awaiting-approval`, `denied`
- `consentUrl`: the URL the user must open (present when status is `pending_configuration` or `awaiting-approval`)

If `status` is `already_configured` or `configured`, stop — the wallet is ready.

## Step 2 — Guide the user to approve

When `status` is `pending_configuration` or `awaiting-approval`:

Show the `consentUrl` to the user:

> To activate your wallet, open this link and approve it. Come back here when you're done.
>
> [consentUrl]

Do not proceed until the user confirms they have approved.
Do not poll automatically. The user must tell you they approved.

## Step 3 — Finalize after approval

When the user says they approved, run:

```bash
lobstercash setup --agent-id <id>
```

The CLI checks the session status automatically. Parse the output:

- `"status": "configured"` — wallet is ready. Continue with the user's original task.
- `"status": "awaiting-approval"` — not approved yet. Show the `consentUrl` again.
- `"status": "denied"` — user denied. Run `lobstercash setup --agent-id <id>` again to generate a fresh consent URL.

## After setup completes

Say: "Wallet ready. Your address is [walletAddress]."

If the user originally asked for something specific (e.g. "buy X", "send tokens"), route to Branch 2 in the main skill file.

If the user did not have a specific task, run `lobstercash store` and present available integrations so they can pick what to do next.

## Anti-Patterns

- **Running setup when the user wants to buy:** Use `request card` or `request deposit` instead — they handle setup automatically.
- **Running `lobstercash setup` more than once without user interaction:** Wait for the user to confirm approval between calls.
- **Asking the user for their wallet address or private key:** The CLI generates and manages keys locally.
- **Polling for approval:** The user must tell you they approved. Do not auto-poll.
