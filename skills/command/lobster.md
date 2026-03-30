---
description: >
  Use this skill when the user wants to make a purchase, send crypto, pay for an API, check their balance, create a virtual card, deposit funds, browse available services, or sign a transaction. Trigger on any spending, payment, or wallet intent — including "buy", "pay", "send money", "check balance", "virtual card", "deposit funds", "what can I buy", or "set up payments" — even without explicitly mentioning "lobster", "wallet", or "Solana".
---

Load the Lobster Cash CLI skill and help with wallet operations and payments.

## Workflow

### Step 1: Load lobster-cash skill

```
skill({ name: 'lobstercash' })
```

### Step 2: Ensure an agent exists

```bash
lobstercash agents list
```

If no agents exist, register one with a descriptive name, description, and image URL:

```bash
lobstercash agents register --name "<descriptive name>" --description "<what the agent does>" --image-url "<avatar url>"
```

### Step 3: Check agent status

```bash
lobstercash status
```

This checks wallet setup, authorization, balances, and payment readiness in one call.

### Step 4: Identify task type from user request

Use the Decision Tree in SKILL.md to select the relevant reference file.

### Step 5: Read the reference file

Based on task type, read `references/<topic>.md`.

### Step 6: Execute task

**Key things to verify:**

- An agent exists (`lobstercash agents list`)
- Wallet is configured (`lobstercash status`)
- Balance covers the operation (`lobstercash balance`)
- Confirm with user before sending tokens or requesting cards
- Follow HITL pattern for approval URLs (do not auto-poll)

### Step 7: Summarize

```
=== Lobster Cash Task Complete ===

Topic: <topic>
<brief summary of what was done>
```

<user-request>
$ARGUMENTS
</user-request>
