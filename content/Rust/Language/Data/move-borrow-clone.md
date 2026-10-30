---
title: Move vs Borrow vs Clone
tags: [rust, ownership]
---

# Move vs Borrow vs Clone

These operations answer different questions: **who owns the value**, **who may access it**, and **whether another value should be created**.

| Operation | Ownership | Original remains usable? | Cost |
| --- | --- | --- | --- |
| Move | Transfers | No, for a moved non-`Copy` value | Often transfers a small representation; no required deep copy |
| Borrow | Stays with owner | Subject to borrow restrictions | Reference creation; no element copy |
| `Copy` | Creates an implicit copy | Yes | Copies the value's representation |
| `.clone()` | Creates another value | Yes | Type-dependent; may allocate |

**Complete example:**

```rust
fn main() {
    let original = String::from("Rust");
    let view = &original; // borrow
    assert_eq!(view.len(), 4);

    let independent = original.clone(); // independent String contents
    let moved = original; // original is now unavailable
    assert_eq!(moved, independent);

    let number = 7;
    let copied = number; // i32 implements Copy
    assert_eq!(number, copied); // both usable
}
```

`Copy` is a marker trait for types that can be duplicated implicitly, such as integers and shared references. Types with destruction responsibilities cannot implement `Copy`. `&mut T` is not `Copy`, because copying an exclusive reference would violate exclusivity.

`Clone` is explicit and customizable. Cloning a `String` duplicates its text allocation; cloning an `Rc<T>` or `Arc<T>` creates another handle to the same allocation. Therefore, “clone” does not always mean “deep copy.”

Moving a `String` does not duplicate its heap characters. The old binding becomes unusable, and the new owner will eventually release the allocation.

Related: [[Rust/Language/Data/ownership|Ownership]] · [[Rust/Language/Advanced/smart-pointers|Shared ownership]].

Sources: [Ownership and moves](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html), [Copy](https://doc.rust-lang.org/std/marker/trait.Copy.html), [Clone](https://doc.rust-lang.org/std/clone/trait.Clone.html).
