---
sidebar_label: 'Gasless Onboarding'
sidebar_position: 10
description: 'New users need native gas before they can do anything. LSP25 lets a relayer submit signed calls with no bundler or paymaster contract.'
---

# Onboard Users Without Making Them Buy Gas First

A brand-new EOA can't do anything until it holds native currency, because an EOA signs and pays in the same step. Every popular fix for this — trusted forwarders, ERC-4337 paymasters, third-party relayer SDKs — adds a layer of infrastructure the builder has to run, rent, or trust. [**LSP25**](../../../standards/accounts/lsp25-execute-relay-call.md) puts relay execution directly on the [Universal Profile](../../universal-profile/metadata/read-profile-data.md): a controller signs a payload, a relayer submits it and pays gas, and `executeRelayCall` handles the rest as a native account function — no bundler, no EntryPoint, no separate mempool.

## Before / after

| Architecture                                      | What it requires                                       | Sponsorship model                                      |
| ------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------ |
| ERC-2771 trusted forwarder                        | every recipient contract inherits `ERC2771Context`     | forwarder pays; contracts trust the forwarder          |
| ERC-4337 + paymaster                              | EntryPoint deposit, bundler, `validatePaymasterUserOp` | a paymaster contract sponsors per `UserOperation`      |
| Third-party relayers (Gelato, Biconomy, Defender) | vendor SDK integration                                 | vendor-hosted infrastructure                           |
| LSP25 `executeRelayCall`                          | a signed payload plus a nonce channel                  | a relayer submits directly to the account — no bundler |

## Why sponsorship needs a separate signer and payer

A new user needs native currency before they can do anything with an EOA, because signing and paying are the same act. Any sponsorship model has to add a layer that separates who signs from who pays — and every popular pattern does that by adding infrastructure: a trusted forwarder you have to trust, an ERC-4337 EntryPoint and bundler you have to run or rent, or a third-party relayer with its own SDK and its own vendor lock-in. The cost lands on the builder, not on the spec.

## What people try today

**EIP-2771 trusted forwarders** let contracts extract the real `msg.sender` from calldata. It's cheap, but every protected contract has to be built forwarder-aware and has to trust that specific forwarder. **ERC-4337 plus a paymaster** is powerful but means shipping or renting the full `UserOperation` pool, bundler, and `EntryPoint` stack. **Turnkey relayers** like Gelato, Biconomy, and Defender wrap one of the above into a convenient SDK, at the cost of vendor coupling. **EIP-7702** lets an EOA act like a smart account for the duration of one transaction, but sponsorship on top of it still needs a paymaster.

## How LSP25 keeps it on the account

LSP25 defines `executeRelayCall` directly on the Universal Profile. A controller signs a payload — with a nonce channel for ordering and replay protection — and a relayer submits the transaction and pays the gas. There's no bundler, no `EntryPoint`, no separate mempool: the account contract _is_ the entry point. Because the relayed call still goes through the account, [LSP6](../../../standards/access-control/lsp6-key-manager.md) permissions apply exactly as they would to any other call — the controller signing the relay payload can only authorize what its permissions allow, so the sponsoring relayer can never escalate the controller's authority just by paying for gas.

:::tip Sponsorship without extra trust
A relayer that pays gas through `executeRelayCall` never gains more authority than the signing controller already had — LSP6 permissions are checked on the relayed call, not bypassed by it.
:::

**Related reading:** [ERC-4337's bundler tax](./erc4337-bundler-tax.md) · [Wallet permission scoping](./wallet-permissions.md) · [Building gasless onboarding](../build/gasless-onboarding.md) · [Gasless onboarding patterns](../architecture/gasless-onboarding-patterns.md) · [Execute relay transactions](../../universal-profile/key-manager/execute-relay-transactions.md)
