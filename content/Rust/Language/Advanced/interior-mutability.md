---
title: Interior Mutability
tags: [rust, advanced]
---

# Interior Mutability

Interior mutability allows controlled mutation through a shared reference. It does not remove access rules: a wrapper enforces a suitable contract through restricted operations, runtime checks, or synchronization.

| Type | Mechanism | Typical use |
| --- | --- | --- |
| `Cell<T>` | Move values in/out; `get` requires Copy | Small single-threaded state |
| `RefCell<T>` | Runtime borrow checking | Single-threaded structures needing dynamic borrows |
| `Mutex<T>` | Exclusive lock | Shared mutable state across threads |
| `RwLock<T>` | Multiple readers or one writer | Read-heavy shared state |
| Atomics | Atomic operations with memory ordering | Counters, flags, specialized coordination |

**Complete example:**

```rust
use std::cell::RefCell;

fn main() {
    let names = RefCell::new(vec!["Ada"]);
    names.borrow_mut().push("Lin");
    let view = names.borrow();
    assert_eq!(view.len(), 2);
}
```

`names` has an immutable binding, but its wrapper permits modification. `borrow_mut()` returns an exclusive guard; dropping the guard ends the borrow. In the example, the temporary guard ends after `push`.

Holding `borrow()` while calling `borrow_mut()` on the same `RefCell` panics. `try_borrow_mut()` reports a conflict as a `Result` instead. `RefCell` is not a synchronization primitive and cannot be used as shared cross-thread state without an appropriate design.

`UnsafeCell` is the primitive underpinning interior mutability. Its existence does not make data races legal; safe wrappers still have to uphold Rust's rules.

Related: [[Rust/Language/Foundations/mutability|Binding mutability]] · [[Rust/Language/Advanced/concurrency|Synchronization]].

Sources: [RefCell and interior mutability](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html), [cell module](https://doc.rust-lang.org/std/cell/).
