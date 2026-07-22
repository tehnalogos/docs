---
sidebar_label: 'ERC2535 Diamonds vs. LSP17'
sidebar_position: 7
description: 'ERC-2535 Diamond proxies compared with LSP17 Contract Extension: storage model, upgrade authority, and permissioning for adding functions post-deploy.'
---

# ERC2535 Diamonds vs. LSP17 Contract Extension

Both standards let a deployed contract grow new functions after launch. ERC-2535 Diamonds route calls via `delegatecall` to facet contracts sharing one storage layout, controlled by a permanent `diamondCut` admin authority. [**LSP17 Contract Extension**](../../../standards/accounts/lsp17-contract-extension.md) routes unknown function selectors through a fallback to per-selector extension contracts, registered under [ERC725Y](../../../standards/erc725.md) and gated by [LSP6](../../../standards/access-control/lsp6-key-manager.md) — with no standing upgrade authority over the base contract at all.

## Comparison

| Feature                              | ERC-2535 Diamonds                                      | LSP17 Contract Extension                                                                                                             |
| ------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| Extension mechanism                  | `delegatecall` to a facet, dispatched by selector      | `CALL` (or `DELEGATECALL`) to an extension, dispatched by selector                                                                   |
| Storage                              | shared with facets — strict layout discipline required | extension keeps its own storage (`CALL` flavor)                                                                                      |
| Registration                         | `diamondCut`, called by the diamond owner              | `setData` on [ERC725Y](../../../standards/erc725.md), permissioned via [LSP6](../../../standards/access-control/lsp6-key-manager.md) |
| Upgrade authority over base bytecode | diamond owner can replace any facet, permanently       | none — the base contract is immutable, extensions are purely additive                                                                |
| Permission granularity               | one `diamondCut` function gates everything             | per-selector `setData` grants — independently scoped per extension                                                                   |

## LSP17 doesn't hand out a master key

Diamonds make the base contract itself a router: every call goes through `delegatecall` into a facet sharing the diamond's storage, and both adding and upgrading behavior go through the same `diamondCut` function. That's powerful, and it's also a standing risk — whoever holds `diamondCut` authority can rewrite what the contract does, permanently, at any time.

LSP17 takes the opposite bet. Known functions run as immutable base bytecode. Only _unknown_ selectors hit the fallback and route to a registered extension. Adding behavior is a `setData` call gated by [LSP6](../../../standards/access-control/lsp6-key-manager.md) permissions — scoped per extension, not a single admin lever over the whole contract. There is no path back to rewriting what the base contract already does.

## The right fit for an account, not just a token

For a protocol contract that genuinely needs to patch bugs after launch, Diamonds' upgrade lever is the point — and users accept that authority as the cost. For an **account** contract like a Universal Profile, that tradeoff inverts: users should never have to trust that a diamond owner won't rewrite their account's behavior. [LSP0](../../../standards/accounts/lsp0-erc725account.md) uses LSP17 for exactly this reason — a Universal Profile can register a new signature verifier or a new asset receiver as an extension, with no admin key sitting over the account itself.

:::tip When to reach for LSP17
Extending an account, a wallet, or any contract where users need a guarantee that existing behavior can't be silently rewritten. Reach for Diamonds only when shared storage across facets is a genuine requirement and the permanent `diamondCut` authority is an accepted cost.
:::

**Related reading:** [The contract-extension problem](../problems/contract-extension.md) · [ERC721 vs LSP8](./erc721-vs-lsp8.md)
