---
title: Collections
tags: [rust, collections]
---

# Collections

Collections organize multiple values. Choose one according to access patterns, ordering, and ownership needs.

| Collection | Useful for | Important property |
| --- | --- | --- |
| `Vec<T>` | Indexed sequences | Contiguous, growable storage |
| `HashMap<K, V>` | Key/value lookup | No guaranteed iteration order |
| `BTreeMap<K, V>` | Ordered key/value lookup | Sorted keys; requires ordering |
| `HashSet<T>` | Unique values | Membership checks; no guaranteed order |
| `VecDeque<T>` | Queues | Efficient operations at both ends |

**Complete example:**

```rust
use std::collections::HashMap;

fn main() {
    let mut counts = HashMap::new();
    for word in ["rust", "gpu", "rust"] {
        *counts.entry(word).or_insert(0) += 1;
    }
    assert_eq!(counts.get("rust"), Some(&2));
    assert_eq!(counts.get("missing"), None);
}
```

`entry` gives access to an occupied or vacant slot. `or_insert(0)` inserts only when absent and returns a mutable reference to the stored value. The `*` accesses that value for mutation.

## Ownership and iteration

| Operation | Yields |
| --- | --- |
| `values.iter()` | Shared references |
| `values.iter_mut()` | Mutable references |
| `values.into_iter()` on an owned Vec | Owned elements; consumes the Vec |

`Vec::len()` counts initialized elements; `capacity()` reports available buffer space before another allocation may be necessary. Growing a vector may reallocate and change element addresses. Rust prevents holding ordinary element references across conflicting mutation.

`HashMap::get` returns `Option<&V>` because a key may be absent; it does not move the value out. Methods such as `remove` can return an owned value.

Related: [[Rust/Language/Modeling/option|Option]] · [[Rust/Language/Abstraction/iterators|Iterators]] · [[Rust/Language/Data/arrays-vec-slices|Vec and slices]].

Source: [Standard collections](https://doc.rust-lang.org/std/collections/).
