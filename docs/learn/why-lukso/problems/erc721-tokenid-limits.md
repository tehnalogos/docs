---
sidebar_label: 'Token ID Design'
sidebar_position: 9
description: 'ERC-721 token IDs are a bare uint256 — awkward for hashes or serials. LSP8 uses bytes32 instead, so structured references fit the standard natively.'
---

# Let Token IDs Carry Meaning, Not Just Count

ERC-721's `uint256 tokenId` is cheap and predictable for sequential mints, but it has no room to be anything else — a content hash, an off-chain serial, or a structured reference to another contract all have to be truncated, cast, or tracked in a side mapping to fit. LSP8 uses [`bytes32 tokenId`](../../../standards/tokens/LSP8-Identifiable-Digital-Asset.md) instead. Sequential collections still work by casting a `uint256` straight into the type — nothing about the simple case gets harder. Everything else — hashes, serials, encoded references — fits without a lookup table, because the ID itself is now a typed value instead of a bare counter.

## Before / after

| Use case                                        | ERC-721 `uint256`                          | LSP8 `bytes32`                                                                           |
| ----------------------------------------------- | ------------------------------------------ | ---------------------------------------------------------------------------------------- |
| Sequential mint counter                         | native fit                                 | native fit — cast `uint256` → `bytes32`                                                  |
| Content hash (e.g. `keccak256` of an asset)     | truncated, or tracked in a side mapping    | fits directly, no truncation                                                             |
| Off-chain serial number                         | side mapping required                      | encode directly into the ID                                                              |
| Structured reference (address + selector, etc.) | not representable                          | encode directly into the ID                                                              |
| Marketplace readability                         | a 78-digit number with no semantic content | the same numeric case still works, or a self-describing ID when you choose to encode one |

## Why a bare integer forces a side mapping

`uint256 tokenId` is a deliberate, minimal primitive — cheap and predictable, and for sequential mints it's genuinely the right answer. The cost shows up the moment the ID needs to _mean_ something: a content hash, a serial number, a structured reference to another contract, and a project ends up maintaining side mappings just to translate between the on-chain integer and whatever it's supposed to represent. The downstream consequence is that token IDs stop being self-describing — you read `42` and have to consult the contract, or a backend, to find out what it actually points to.

## What people try today

**Off-chain registries** map `uint256` IDs to richer identifiers in a database. It works, but it couples the asset permanently to that project's infrastructure. **Hash-truncation conventions** cast a 32-byte hash down into a `uint256` — functional, but marketplace UIs still display a 78-digit number with no semantic content a human or a tool can read. **Multiple side mappings** — one per dimension a project wanted to encode — grow storage cost linearly with what should have been a single, well-typed ID.

## How bytes32 removes the ceiling

LSP8 uses `bytes32` token IDs. The integer case still works exactly as before — cast a `uint256` to `bytes32` and the sequential model behaves identically. When an ID needs to carry a content hash, the hash is stored directly. When it needs a structured serial, the serial is encoded directly. When it needs to reference another contract, the address and selector are encoded directly. The ID becomes a typed value instead of a counter pretending to be a name — no side mapping required to know what it is.

:::tip Default to bytes32 unless you need pure sequential IDs
LSP8's `bytes32` costs nothing over `uint256` for a simple counter-based collection, and removes the ceiling the moment an ID needs to encode a hash, serial, or reference.
:::

**Related reading:** [Dynamic NFT metadata](./erc721-dynamic-metadata.md) · [ERC-721's safe transfer gap](./erc721-safe-transfer.md) · [ERC-1155 complexity](./erc1155-complexity.md) · [ERC721 vs. LSP8](../compare/erc721-vs-lsp8.md)
