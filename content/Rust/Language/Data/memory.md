---
title: Stack, Heap, and Drop
tags: [rust, memory]
---

# Stack, Heap, and Drop

The **stack** supports function-local storage with last-in, first-out call frames. The **heap** supports allocations whose size and lifetime are managed separately from those frames. These are useful implementation models; optimization can remove or rearrange storage.

```text
Local String value                 Heap allocation
┌──────────────────────┐           ┌──────────────────┐
│ pointer ────────────────────────►│ R  u  s  t       │
│ length: 4            │           │ spare capacity…  │
│ capacity: at least 4 │           └──────────────────┘
└──────────────────────┘
Conceptual fields; not a guaranteed field order or address layout.
```

A `String` owns its heap allocation. A `Vec<T>` similarly tracks a buffer, length, and capacity. The handle itself can be stored in a local, another allocation, or a larger structure.

Arrays store their elements inline wherever the array is stored: `[T; N]` is not inherently a stack-only type. A `Box<[T; N]>` owns an array in a heap allocation.

## Destruction

When a value is dropped, Rust runs its destructor, then drops its fields. Owners such as `String` release resources through this mechanism. `std::mem::drop(value)` explicitly consumes and drops a value early.

**Complete example:**

```rust
struct Notice;

impl Drop for Notice {
    fn drop(&mut self) {
        println!("released");
    }
}

fn main() {
    let notice = Notice;
    drop(notice); // prints "released" here
    println!("after");
}
```

Destructors can release files and locks as well as memory. They are not guaranteed to run on process abort or forced termination, so they are not a substitute for durable storage protocols.

Ownership describes responsibility and access, not physical location. Moving an owner does not necessarily move the allocation it owns.

Related: [[Rust/Language/Advanced/smart-pointers|Smart pointers]] · [[Rust/Language/Data/ownership|Ownership]].

Sources: [Stack and heap](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap), [Drop](https://doc.rust-lang.org/std/ops/trait.Drop.html).
