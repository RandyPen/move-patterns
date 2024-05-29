# ID Pointer

|||
|-|-|
| **Name** | ID Pointer |
| **Origin** | [Sui Project](https://docs.sui.io/concepts/sui-move-concepts/patterns/id-pointer) |
| **Example** | [example.move](https://github.com/MystenLabs/sui/blob/main/examples/move/hero/sources/example.move#L70) |
| **Depends on** | None |
| **Known to work on** | Move |

## Summary

The following example implements basic `Lock` and `Key` mechanics on Sui where `Lock<T>` is a shared object that can contain any object, and `Key` is an owned object, which is required to get access to the contents of the lock.

`Key` is linked to its `Lock` using an `ID` field. This check allows off-chain discovery of the target, as well as splits the dynamic transferable capability and the 'static' contents. Another benefit of this approach is that the target asset is always discoverable, while you can wrap its `Key` into another object (such as a marketplace listing).

**ID pointer** is a technique that separates the main data (an object) and its accessors/capabilities by linking the latter to the original. There's a few different ways you can use this pattern:

- Issuing transferable capabilities for shared objects (for example, a TransferCap that changes 'owner' field of a shared object)
- Splitting dynamic data and static (for example, an NFT and its collection information)
- Avoiding unnecessary type linking (and witness requirement) in generic applications (LP token for a `LiquidityPool`)

## Examples

The following example implements basic Lock and Key mechanics on Sui where Lock<T> is a shared object that can contain any object, and Key is an owned object, which is required to get access to the contents of the lock.

Key is linked to its Lock using an ID field. This check allows off-chain discovery of the target, as well as splits the dynamic transferable capability and the 'static' contents. Another benefit of this approach is that the target asset is always discoverable, while you can wrap its Key into another object (such as a marketplace listing).

```rust
// Copyright (c) 2022, Mysten Labs, Inc.
// SPDX-License-Identifier: Apache-2.0

module examples::lock_and_key {
    use sui::object::{Self, ID, UID};
    use sui::transfer;
    use sui::tx_context::{Self, TxContext};
    use std::option::{Self, Option};

    /// Lock is empty, nothing to take.
    const ELockIsEmpty: u64 = 0;

    /// Key does not match the Lock.
    const EKeyMismatch: u64 = 1;

    /// Lock already contains something.
    const ELockIsFull: u64 = 2;

    /// Lock that stores any content inside it.
    struct Lock<T: store + key> has key {
        id: UID,
        locked: Option<T>
    }

    /// A key that is created with a Lock; is transferable
    /// and contains all the needed information to open the Lock.
    struct Key<phantom T: store + key> has key, store {
        id: UID,
        for: ID,
    }

    /// Returns an ID of a Lock for a given Key.
    public fun key_for<T: store + key>(key: &Key<T>): ID {
        key.for
    }

    /// Lock some content inside a shared object. A Key is created and is
    /// sent to the transaction sender. For example, we could turn the
    /// lock into a treasure chest by locking some `Coin<SUI>` inside.
    ///
    /// Sender gets the `Key` to this `Lock`.
    public fun create<T: store + key>(obj: T, ctx: &mut TxContext) {
        let id = object::new(ctx);
        let for = object::uid_to_inner(&id);

        transfer::share_object(Lock<T> {
            id,
            locked: option::some(obj),
        });

        transfer::transfer(Key<T> {
            for,
            id: object::new(ctx)
        }, tx_context::sender(ctx));
    }

    /// Lock something inside a shared object using a Key. Aborts if
    /// lock is not empty or if key doesn't match the lock.
    public fun lock<T: store + key>(
        obj: T,
        lock: &mut Lock<T>,
        key: &Key<T>,
    ) {
        assert!(option::is_none(&lock.locked), ELockIsFull);
        assert!(&key.for == object::borrow_id(lock), EKeyMismatch);

        option::fill(&mut lock.locked, obj);
    }

    /// Unlock the Lock with a Key and access its contents.
    /// Can only be called if both conditions are met:
    /// - key matches the lock
    /// - lock is not empty
    public fun unlock<T: store + key>(
        lock: &mut Lock<T>,
        key: &Key<T>,
    ): T {
        assert!(option::is_some(&lock.locked), ELockIsEmpty);
        assert!(&key.for == object::borrow_id(lock), EKeyMismatch);

        option::extract(&mut lock.locked)
    }

    /// Unlock the Lock and transfer its contents to the transaction sender.
    public fun take<T: store + key>(
        lock: &mut Lock<T>,
        key: &Key<T>,
        ctx: &mut TxContext,
    ) {
        transfer::public_transfer(unlock(lock, key), tx_context::sender(ctx))
    }
}
```

*Example for Sui Move is taken from the [Sui Doc](https://docs.sui.io/concepts/sui-move-concepts/patterns/id-pointer).*