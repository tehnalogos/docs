---
sidebar_label: '🔐 Safe to Universal Profile'
sidebar_position: 5
description: 'Step-by-step guide to migrating custody from a Gnosis Safe multisig to a LUKSO Universal Profile: mapping signers to LSP6 controller permissions.'
---

# 🔐 Migrate from Safe to Universal Profile

Moving custody from a Safe (Gnosis Safe) multisig to a [**Universal Profile**](../universal-profile/metadata/read-profile-data.md) maps the multisig-signer model onto [LSP6 Key Manager](../../standards/access-control/lsp6-key-manager.md) controllers. Each Safe owner becomes an LSP6 controller address with its own permission bitfield (`CALL`, `SETDATA`, `TRANSFERVALUE`, `ADDCONTROLLER`, `SIGN`, and more) — narrower and more expressive than a flat multisig owner list.

:::info Estimate
Roughly a day for a small Safe. Longer if the Safe custodies many distinct asset types, since each needs its own transfer step.
:::

## When to migrate

Migrate when per-controller permission scoping in a standard vocabulary — the LSP6 bitfield plus `AllowedCalls` and `AllowedERC725YDataKeys` — profile-native metadata storage, or [LSP25](../../standards/accounts/lsp25-execute-relay-call.md) relay execution on the account itself is what you need. Stay on Safe when an m-of-n multisig threshold is the exact primitive your product encodes — that pattern doesn't map one-to-one onto LSP6 and needs an explicit recovery-contract layer to express instead.

## Step 1 — model the Safe as permissions

Safe's "3 of 5" threshold doesn't exist directly in LSP6 — controllers are individual, not aggregated by a vote. Two patterns work:

- **Recovery contract** — deploy a contract that enforces the threshold itself, and register that contract as the controller holding `EDITPERMISSIONS` on the profile. Day-to-day controllers handle daily operations; the recovery contract handles ownership-level changes.
- **Single day-to-day controller + cold multisig** — flatten daily operations to one controller, and keep the Safe (or a new threshold contract) as the cold recovery layer behind it.

## Step 2 — deploy the Universal Profile

Deploy with the standard `lsp-factory.js` (or equivalent) deployment script. Set [LSP3](../../standards/metadata/lsp3-profile-metadata.md) profile metadata, then add controllers per the design from Step 1.

## Step 3 — transfer assets

For each asset class:

- **Native LYX** — `Safe.execTransaction` → `profile.execute(0, profile, value, "")`
- **ERC20 / LSP7 tokens** — `Safe` → `token.transfer(profile, balance)`
- **NFTs** — `Safe` → `token.safeTransferFrom(safe, profile, id)` (or the LSP8 equivalent)

If the Safe holds many distinct assets, write a sweep contract that batches the outbound transfers instead of sending them one by one.

## Step 4 — update integrations

Identify every protocol that references the Safe's address directly: vesting contracts, DAO memberships, subscriptions, allowance grants. Addresses don't migrate on their own — every external reference needs to be re-pointed to the new profile address.

## Step 5 — sunset the Safe

Once nothing material remains in the Safe, sweep out any remaining gas dust and treat it as historical. Leave the contract deployed — destroying multisig contracts is unsupported and risky.

## Gotchas

- Multisig threshold semantics don't map one-to-one to LSP6 — model it as a recovery-controller contract that enforces the threshold, then register that contract as an LSP6 controller with `EDITPERMISSIONS`.
- Asset transfer is many separate transactions unless the Safe owns assets through a sweep contract that batches outbound moves.
- Anything connected to the Safe's address — vesting schedules, allowance grants, on-chain memberships — needs to be re-pointed to the new profile address; addresses don't migrate, only the references you update do.
- Safe modules have their own permission shape; LSP6 controllers plus [LSP17](../../standards/accounts/lsp17-contract-extension.md) extensions are the closest LSP-side equivalents. Map each module to its closest LSP primitive deliberately rather than assuming a 1:1 translation.

## Verify the migration

- All assets transferred to the Universal Profile.
- Old Safe paused — no longer holds material assets.
- Controllers and permissions configured on the new profile.
- Recovery policy in place, with at least one cold controller holding `EDITPERMISSIONS`.
- Off-chain integrations updated to the new address.

**Related reading:** [EOA vs Universal Profile](../why-lukso/compare/eoa-vs-universal-profile.md) · [Wallet permission scoping](../why-lukso/problems/wallet-permissions.md)
