---
sidebar_position: 3
title: XODE Transaction Fees
description: Weight, storage rent, and storage deposit limits for smart contracts on XODE
---

# XODE Transaction Fees

## Weight

As with Substrate, `pallet-contracts` uses Weight V2 to charge execution fees. Weight is composed of two parts:

- refTime: computational time used for execution, in picoseconds.
- proofSize: size of data included in the proof of validity so the Relay Chain can verify state changes, in bytes. Accessing storage increases gas fees due to larger proofs.

Information: Gas = Weight = (refTime, proofSize). See Substrate documentation on transaction weight.

## Storage Rent

Storage rent (Automatic Deposit Collection) protects the chain from on-chain storage spam. It ensures callers have financial stake when storing data and incentivizes removing unused data:

- Users are charged per-byte stored on-chain. The fee is moved from the user's free balance to the contract's reserved balance.
- Contracts cannot spend the reserved balance directly (though they can expose functions to prune storage so callers can reclaim deposits).
- Any user can reclaim rent by removing on-chain data, depending on contract interfaces.

## Storage Rent Calculation

This fee uses two price constants:

- Per Storage Byte: 0.000001 XON
- Per Storage Item: 0.00004 XON

### Calculation

When storing a new key/value in a `Mapping`, one `DepositPerItem` is charged. The byte length of the value is also charged: `bytes_length × DepositPerByte`.

Example: storing a new entry in `Mapping<u32, AccountId>` (where `AccountId` is 32 bytes) costs `DepositPerItem + 32 × DepositPerByte`.

## For users

- The first call to a dApp is usually more expensive because it creates new storage entries (often keyed by the user's `AccountId`).
- Subsequent calls that only modify existing entries are cheaper (or free aside from execution weight) and only charge for additional bytes added; reducing size returns rent to the caller.
- Users can increase free balance by removing on-chain data when contracts expose such functionality.

If a user wants to reclaim deposits, they must remove on-chain data. This requires the contract to expose functions that remove storage (e.g., `remove_mapping_entry`).

## For smart-contract developers

Expose storage cleanup functions so users can reclaim reserved balances. If not provided, users cannot remove the storage and recover deposits (funds remain reserved on the contract account).

## StorageDepositLimit

Contract calls include a `StorageDepositLimit` argument, which caps the maximum storage rent charged for a single call.

Important: If `StorageDepositLimit` is set to `None`, contracts may charge an arbitrary amount from the caller's account. Always dry-run to estimate deposits and set a limit, particularly in frontends and generic UIs.

Users are responsible for both gas limit and storage deposit limit.

## Contract example on XODE

```rust
#[ink::contract]
mod rent {
    use ink::storage::Mapping;

    #[ink(storage)]
    pub struct Rent {
        map: Mapping<AccountId, u32>,
        int: u32,
        bool: bool,
    }

    impl Rent {
        #[ink(constructor)]
        pub fn new() -> Self {
            Self { map: Default::default(), int: 0, bool: false }
        }

        #[ink(message)]
        pub fn update_32(&mut self, i: u32) {
            self.int = i
        }

        #[ink(message)]
        pub fn flip_bool(&mut self) {
            self.bool = !self.bool
        }

        #[ink(message)]
        pub fn add_mapping_entry(&mut self) {
            let caller = self.env().caller();
            // Insert one item to storage. fee = 1 * PricePerItem (0.0004 XON)
            // Value is u32 (4 bytes). fee = 4 * PricePerByte (0.00002 XON) = 0.00008 XON
            // Total fee = 0.00408 XON
            self.map.insert(caller, &1u32);
        }

        #[ink(message)]
        pub fn remove_mapping_entry(&mut self)  {
            let caller = self.env().caller();
            // Clears the value at key from storage.
            // Remove one item from storage. fee = 1 * PricePerItem (0.0004 XON)
            // Remove the value (u32, 4 bytes). fee = 4 * PricePerByte (0.00002 XON) = 0.00008 XON
            // Total reserve repatriated by caller = 0.00408 XON
            self.map.remove(caller);
        }

        #[ink(message)]
        pub fn remove_entry_account_id(&mut self, who: AccountId)  {
            // Clears the value at key from storage.
            // Remove one item from storage. fee = 1 * PricePerItem (0.0004 XON)
            // Remove the value (u32, 4 bytes). fee = 4 * PricePerByte (0.00002 XON) = 0.00008 XON
            // Total reserve repatriated by caller = 0.00408 XON
            self.map.remove(who);
        }
    }
}
```

### add_mapping_entry

The reserved balance (moved from the caller's free balance to the contract's reserved balance) will be:

- Insert one item to storage: `1 × PricePerItem (0.0004 XON)`
- Value is `u32` (4 bytes): `4 × PricePerByte (0.00002 XON) = 0.00008 XON`
- Total fee = `0.00408 XON`

### remove_mapping_entry

The balance repatriated (moved from the contract's reserved to the caller's free balance) will be:

- Remove one item from storage: `1 × PricePerItem (0.0004 XON)`
- Remove value `u32` (4 bytes): `4 × PricePerByte (0.00002 XON) = 0.00008 XON`
- Total reserve repatriated by caller = `0.00408 XON`

### remove_entry_account_id

The caller receives the repatriated balance (not necessarily the original payer): `0.00408 XON`.

### flip_bool & update_32

No rent fees; they only update existing values and do not add new on-chain storage.

---

See also: [XODE Staking and Consensus](./xode-stacking.md)


