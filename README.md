# index-set

bitset implementation with support for atomic operations

[![Crates.io](https://img.shields.io/crates/v/index-set.svg)](https://crates.io/crates/index-set)
[![Documentation](https://docs.rs/index-set/badge.svg)](https://docs.rs/index-set)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

## Why use `index-set`?

In our use case, We needed to track the online/offline status of millions of users with minimal memory usage and lightning-fast lookup performance. Our ideal solution required the following:

- Reuses identifiers when they are removed from the set.
  When an identifier is removed, it is recycled for future use.

- Atomic and thread-safe operations.
- Constant-time performance: Insertion, removal, and lookup operations must all be `O(1)`.
- Compact memory usage, Each identifier is represented by a bit in the memory.
  For example, `1` megabyte of memory can store `8` millions (`8,388,608`) unique identifiers.

- Identifiers are unique and as small as possible.

## Example

Add this to your `Cargo.toml` file:

```toml
[dependencies]
index-set = "0.1"
```

Here is a simple example of how to use `AtomicBitSet`:

```rust
use index_set::{AtomicBitSet, slot_count, BitSet, SharedBitSet};

// Create a new AtomicBitSet with memory size of 1 kilobyte
static BIT_SET: AtomicBitSet<{ slot_count::from_kilobytes(1) }> = AtomicBitSet::new();

fn main() {
    assert_eq!(BIT_SET.set_next_free_bit(), Some(0));

    BIT_SET.insert(2);
    assert_eq!(BIT_SET.set_next_free_bit(), Some(1));
    assert_eq!(BIT_SET.set_next_free_bit(), Some(3));

    BIT_SET.remove(1);
    assert_eq!(BIT_SET.has(1), false);
    assert_eq!(BIT_SET.set_next_free_bit(), Some(1));

    assert_eq!(BIT_SET.size(), 4);

    // it can hold up to 8192 unique identifiers.
    assert_eq!(BIT_SET.capacity(), 8192);
}
```

Here is basic usage of `BitSet` and `BitSetMut` traits.

```rust
use index_set::{BitSet, BitSetMut, slot_count};

fn main() {
    let mut bitset = [0_u32; 2];

    bitset.insert(42);
    assert_eq!(bitset.has(42), true);
    assert_eq!(bitset.remove(42), Some(true));

    assert_eq!(bitset.size(), 0);
    assert_eq!(bitset.capacity(), 64);
}
```