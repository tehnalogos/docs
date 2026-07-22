---
sidebar_label: '🎒 ERC1155 to LSP7 + LSP8'
sidebar_position: 4
description: 'Step-by-step guide to migrating an ERC1155 multi-asset contract to LSP7 and LSP8 on LUKSO by splitting the collection along fungible vs identifiable shape.'
---

# 🎒 Migrate ERC1155 to LSP7 + LSP8

ERC1155 packs fungible and identifiable token IDs into one contract, with the type signaled by convention in the ID bits. Migrating to LUKSO splits that surface along asset semantics instead: fungible IDs become an [**LSP7**](../../standards/tokens/LSP7-Digital-Asset.md) contract, identifiable IDs become an [**LSP8**](../../standards/tokens/LSP8-Identifiable-Digital-Asset.md) contract. Both dispatch transfer notifications through the same [LSP1](../../standards/accounts/lsp1-universal-receiver.md) `universalReceiver(typeId, data)` hook — receivers branch on `typeId`, not on a per-collection ID-bit convention.

:::info Estimate
Roughly a week for a typical game or edition contract. Effort scales with how much of your existing code assumed the ERC1155 type-bit decoding convention rather than reading semantics from the standard itself.
:::

## When to migrate

Migrate when downstream code needs to dispatch on asset shape — fungible vs. identifiable — by interface, without a per-collection decoding rule. Keep ERC1155 when atomic cross-ID batch transfers are a primitive your protocol depends on, or when per-ID semantics are baked into your contract's state machine in a way that resists splitting.

## Step 1 — classify every token ID

For each token ID (or ID range) in the existing ERC1155 contract, decide: is it **fungible** (interchangeable units, summable) or **identifiable** (each unit unique)? Fungible → LSP7. Identifiable → LSP8. A semi-fungible edition (e.g. 100 prints of the same piece) can work as LSP7 with `decimals` set to `0`.

## Step 2 — deploy the two contracts

One LSP7 contract for the fungible inventory, one LSP8 contract for the identifiable inventory. [LSP4](../../standards/tokens/LSP4-Digital-Asset-Metadata.md) metadata lives independently on each.

## Step 3 — port the holders

Snapshot ERC1155 balances. For LSP7 IDs, mint the equivalent amounts. For LSP8 IDs, mint `bytes32`-encoded token IDs. This is typically two coordinated mint scripts, sometimes merkle-claimed for large holder sets.

## Step 4 — port the integrations

Every place that implemented `IERC1155Receiver` needs [LSP1](../../standards/accounts/lsp1-universal-receiver.md) `universalReceiver` support instead. The `typeId` on each call lets a receiver branch on "is this an LSP7 transfer?" vs. "is this an LSP8 transfer?" — one hook function handles both, rather than separate single/batch receiver interfaces.

## Step 5 — sunset the old contract

Once the new contracts are live and every integration has migrated, pause the ERC1155 contract. Keep it deployed for historical reference — don't redeploy over it.

## Gotchas

- Two contracts instead of one — deployment cost doubles, and so does the indexing surface.
- Batch transfers are now per-standard, not cross-standard — a batch can't mix LSP7 and LSP8 items in one call.
- Holders with mixed ERC1155 IDs need a coordinated mint across both new contracts.
- Marketplaces that supported your ERC1155 contract won't automatically pick up the new LSP7 + LSP8 contracts — plan for marketplace re-listing.

## Verify the migration

- Every ERC1155 token ID is mapped to either an LSP7 amount or an LSP8 token ID.
- Holder balances are reproduced identically across both new contracts.
- Batch transfer behavior is tested per standard.
- LSP1 receivers correctly handle both LSP7 and LSP8 `typeId`s.

**Related reading:** [ERC1155 vs LSP7 + LSP8](../why-lukso/compare/erc1155-vs-lsp7-lsp8.md) · [ERC1155's complexity problem](../why-lukso/problems/erc1155-complexity.md)
