---
sidebar_label: 'ERC-4337 Bundler Tax'
sidebar_position: 6
description: 'ERC-4337 account abstraction runs on a parallel mempool of bundlers and paymasters. LSP0 + LSP6 + LSP25 put relay execution on the account itself.'
---

# Skip the Bundler Mempool for Account Abstraction

ERC-4337 delivers account abstraction by building an entirely parallel transaction system on top of the one that already exists — UserOperations, EntryPoint contracts, bundlers, paymasters, signature aggregators. Every team that adopts it either runs that infrastructure or rents it from a third party, and debugging happens on a separate plane from ordinary transactions: block explorers show `handleOps`, not your function call. A [**Universal Profile**](../../universal-profile/metadata/read-profile-data.md) gets the same account-abstraction outcomes — sponsored execution, scoped permissions, contract-native signing — by composing [LSP0](../../../standards/accounts/lsp0-erc725account.md), [LSP6](../../../standards/access-control/lsp6-key-manager.md), [LSP20](../../../standards/accounts/lsp20-call-verification.md), and [LSP25](../../../standards/accounts/lsp25-execute-relay-call.md) directly on the account contract — no EntryPoint hop, no bundler, no UserOperation envelope.

## Before / after

| Layer                              | ERC-4337                                                  | LSP stack (Universal Profile)                          |
| ---------------------------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| Entry point                        | separate `EntryPoint` contract                            | the account contract itself                            |
| Mempool                            | alt-mempool for UserOperations                            | ordinary chain mempool                                 |
| Sponsorship                        | paymaster contract + EntryPoint deposit                   | LSP25 `executeRelayCall`, relayer submits directly     |
| Validation                         | pluggable validator module per account                    | LSP6 permission check + LSP20 inline call verification |
| `msg.sender` at downstream targets | the account, after EntryPoint routing                     | the account, directly                                  |
| Tooling required                   | bundler infrastructure — Pimlico, Alchemy, Biconomy, etc. | standard RPC plus a relayer service                    |

## Why the parallel mempool is a real cost

ERC-4337 ships account abstraction _over_ a transaction system that was designed for EOAs, and the price is a second infrastructure stack most product teams don't want to run or rent: `UserOperation`s, `EntryPoint` contracts, bundlers, paymasters, signature aggregators. The value model still routes through the EntryPoint even though the account itself is a smart contract, so debugging, tracing, and gas accounting all sit on a separate plane from regular transaction flow.

## What people try today

**Safe plus a relayer** is a solid model — Safe is a smart account contract, and a relayer simply submits the multisig's `exec` call, with no `UserOperation` and no bundler involved. It's just less standardized than ERC-4337. **EIP-7702** lets an EOA temporarily delegate to a contract for one transaction, getting smart-account semantics without becoming a contract permanently, though sponsorship still needs a paymaster on top. **Custodial backends** sign user actions server-side for the simplest possible UX, at the cost of the weakest custody story. **App-specific forwarders** built on EIP-2771 are straightforward but narrow to a single product and are hard to share across protocols.

## How the LSP stack avoids the extra hop

A Universal Profile is a contract account composed from LSP0 (the ERC-725 account contract itself), LSP6 (Key Manager permissions on every controller), LSP25 (`executeRelayCall` for sponsored execution), and LSP20 (inline call verification). There is no bundler and no EntryPoint: a controller signs, a relayer submits, and the profile validates via LSP6 and executes directly. Because the relayed call goes through the account, LSP6 permissions still apply — the controller signing the relay payload can only authorize what its permissions allow.

The trade-off is worth stating plainly. ERC-4337 decouples the account from its validator, so you can swap validators independently of the account or mix them per UserOperation — that decoupling is the model's real strength, and its cost shows up as extra hops and a non-standard, per-wallet permission shape. The LSP stack fuses the account and its policy instead: LSP6 is the one permission vocabulary, LSP20 verifies calls inline, and LSP25 handles sponsored execution as a plain function on the account. Extending behavior later means adding [LSP17 extensions](./contract-extension.md) or composing new LSPs, not hot-swapping a validator module.

:::tip Fewer hops, one vocabulary
If a product wants sponsored transactions and scoped permissions without operating bundler infrastructure, LSP0 + LSP6 + LSP20 + LSP25 deliver it on the account itself — the trade is a fused account/policy model instead of ERC-4337's swappable validators.
:::

**Related reading:** [Gasless onboarding without a paymaster](./gasless-onboarding.md) · [Wallet permission scoping](./wallet-permissions.md) · [ERC-4337 vs. the LSP stack](../compare/erc4337-vs-lsp-stack.md) · [ERC-4337 vs. EIP-7702](../compare/erc4337-vs-eip7702.md) · [The 4337 extension for Universal Profiles](../../universal-profile/advanced-guides/4337-extension.md)
