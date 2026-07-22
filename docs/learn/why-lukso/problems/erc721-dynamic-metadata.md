---
sidebar_label: 'Dynamic NFT Metadata'
sidebar_position: 7
description: 'ERC-721 tokenURI returns one string with no integrity guarantee. LSP8 stores per-token metadata as typed, verifiable ERC725Y keys.'
---

# Make NFT Metadata Verifiable, Not Just Hosted

ERC-721's `tokenURI(tokenId)` returns a single string, and the contract has no opinion about what that string points to, who can change it, or whether a marketplace's cached copy is still accurate. LSP8 replaces the single URI with typed [ERC725Y](../../../standards/erc725.md) key-value storage per token: `getDataForTokenId(tokenId, key)` returns a typed value directly, and [LSP4](../../../standards/tokens/LSP4-Digital-Asset-Metadata.md)'s `VerifiableURI` type pairs an off-chain pointer with an on-chain hash, so a silent swap of the underlying file becomes detectable instead of invisible.

## Before / after

| Property             | ERC-721 `tokenURI`                                                 | LSP8 + LSP4                                                                                                     |
| -------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| Storage shape        | one string per token                                               | typed [ERC725Y](../../../standards/erc725.md) key-value pairs per token                                         |
| Integrity            | none — trust whoever hosts the JSON                                | `VerifiableURI` carries an on-chain hash of the payload                                                         |
| Update signal        | ERC-4906 `MetadataUpdate` event (says something changed, not what) | a `setData` call, readable directly on-chain                                                                    |
| Update authorization | contract-specific, often owner-only                                | scoped through whatever [LSP6](../../../standards/access-control/lsp6-key-manager.md) permissions you configure |
| Read pattern         | fetch the URI, fetch the JSON, trust the cache                     | `getDataForTokenId(tokenId, key)`, typed return, hash-checkable                                                 |

## Why one cached string can't carry dynamic state

`tokenURI(tokenId)` returns one string, and the contract has no opinion on what changes, when, or by whom. If the metadata is meant to be dynamic, the actual truth lives wherever the JSON is hosted — and every marketplace, wallet, and explorer caches its own copy on its own schedule. "Refresh metadata" buttons exist precisely because there's no standard signal for what changed; ERC-4906 added a `MetadataUpdate` event, but consumers still have to go ask the server what the new state actually is.

## What people try today

**Mutable IPFS pointers** are convenient, but the JSON is still off-chain and trust-anchored to whoever holds the ability to repin the CID. **Server-rendered metadata APIs** work at scale but couple the NFT permanently to the issuer's infrastructure — the collection falls over when the server does. **ERC-4906's `MetadataUpdate` event** signals that something changed without saying what changed or carrying the new state. **Fully on-chain SVG generators** give the strongest integrity guarantee, but are expensive and awkward for rich media — a good answer for a narrow set of collections, not a general one.

## How LSP8 and LSP4 make metadata a verifiable graph

LSP8 assets store per-token metadata through [ERC725Y](../../../standards/erc725.md) data keys instead of a single `tokenURI` string. LSP4 defines the metadata conventions — name, symbol, JSON schema — and its `VerifiableURI` type lets a key point to an off-chain payload _with a hash_, so the chain enforces what that off-chain blob is required to contain. Apps read `getDataForTokenId(tokenId, key)` and get a typed value back directly. Dynamic updates become `setData` calls, gated by whatever account permissions the issuer wants to configure. The metadata stops being a URL to trust and becomes a typed key-value graph you can verify.

:::tip Verifiable beats cached
For any collection where metadata legitimately changes after mint, LSP8's `VerifiableURI` gives consumers a hash to check against — instead of asking them to trust that the cache is current.
:::

**Related reading:** [ERC-721's safe transfer gap](./erc721-safe-transfer.md) · [ERC-721 token ID limits](./erc721-tokenid-limits.md) · [Building dynamic NFTs](../build/dynamic-nfts.md) · [ERC721 vs. LSP8](../compare/erc721-vs-lsp8.md)
