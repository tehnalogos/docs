---
sidebar_label: 'ERC20 Approval Risk'
sidebar_position: 4
description: 'ERC-20 approve grants standing, often unlimited, allowances. LSP7 moves scope enforcement to LSP6 account permissions instead.'
---

# Stop Granting Standing Allowances to Token Contracts

ERC-20's `approve(spender, amount)` asks a token contract to trust a spender indefinitely — most dApps request max-uint to avoid a second prompt, and that allowance sits live on-chain until someone manually revokes it. LSP7's `authorizeOperator` is still amount-scoped, exactly like `approve` — LUKSO isn't pretending otherwise. What changes is _where the policy lives_: on a [Universal Profile](../../universal-profile/metadata/read-profile-data.md), the controller calling `authorizeOperator` is itself bound by [LSP6 Key Manager](../../../standards/access-control/lsp6-key-manager.md) permissions, so enforcement moves from a forever-promise buried in the token contract to a revocable grant on the account itself.

## Before / after

| Mechanism            | ERC-20                                                | LSP7 + LSP6                                                             |
| -------------------- | ----------------------------------------------------- | ----------------------------------------------------------------------- |
| Grant call           | `approve(spender, amount)`                            | `authorizeOperator(operator, amount, data)`                             |
| Common default       | dApps request max-uint to skip re-prompting           | still amount-scoped by convention — not fixed by LSP7 alone             |
| Who enforces scope   | the token contract, forever, until revoked            | the account, via LSP6 permissions on the granting controller            |
| Revocation           | separate `approve(spender, 0)` transaction, per token | revoke the controller's LSP6 permission — one transaction, account-wide |
| Session-style access | not supported natively                                | grant a scoped session controller, then revoke it when the session ends |

## Why approve/transferFrom splits intent from execution

`approve` sets an allowance; the spender calls `transferFrom` later, whenever it wants, for any amount up to that allowance. The user signs intent once and never sees the individual executions, so a wallet can't meaningfully explain what it's authorizing beyond a number. Because most dApps request max-uint to avoid asking twice, every approved spender becomes a standing risk — if that spender contract is later upgraded, exploited, or misconfigured, the allowance is already sitting there waiting to be used.

## What people try today

**Revoke flows** like revoke.cash and wallet allowance dashboards are reactive — they require the user to notice and act, after the allowance has already lived on-chain, potentially for months. **EIP-2612 Permit** moves the approval from a transaction into a signature, which is cheaper to give but is still blanket authority that still trusts the spender contract not to misuse it. **Permit2** scopes allowances per transaction and per contract, which helps materially, but only when both the token and the integrating app support it. **Approve-and-call wrapper contracts** add a router in front of the token, which fragments the UX across two contracts without touching the underlying ERC-20 model.

## How LSP6 moves the policy to the account

LSP7's `authorizeOperator` doesn't magically fix amount-scoped allowances — that part of the risk is unchanged. What's different is that on a Universal Profile, the controller invoking `authorizeOperator` is itself governed by LSP6: allowed calls, allowed standards, allowed [ERC725Y](../../../standards/erc725.md) data keys, value limits, all revocable per controller. Instead of asking the token contract to police every future `transferFrom` forever, the account decides what each app controller may invoke, and can cut that controller off in a single transaction — without ever touching a token-level allowance.

:::tip Honest framing, real change
LSP7 operators are still amount-scoped — that isn't a magic fix, and LUKSO doesn't claim otherwise. The meaningful shift is that enforcement moves to the account: a session controller can be granted and revoked in one transaction each, on-chain, instead of trusting a token contract's allowance forever.
:::

**Related reading:** [Wallet permission scoping](./wallet-permissions.md) · [ERC-20's missing transfer hooks](./erc20-transfer-hooks.md) · [ERC-20 explained](../erc-explainers/erc-20.md) · [ERC20 vs. LSP7](../compare/erc20-vs-lsp7.md)
