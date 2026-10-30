---
title: Box, Rc, Arc, Deref, and Drop
tags: [rust, advanced, memory]
---

# Box, Rc, Arc, Deref, and Drop

Smart pointers are values that manage access to another value, often with ownership or cleanup behavior.

| Type | Ownership | Thread use | Purpose |
| --- | --- | --- | --- |
| `Box<T>` | One owner | Depends on T | Heap allocation, recursive types, owned trait objects |
| `Rc<T>` | Shared, reference-counted | Single thread | Shared ownership without atomic counting |
| `Arc<T>` | Shared, atomic reference-counted | Depends on T | Share ownership across threads |
| `Weak<T>` | Non-owning reference to Rc/Arc allocation | Matches its family | Break ownership cycles |

**Complete example:**

```rust
use std::rc::Rc;

fn main() {
    let first = Rc::new(String::from("shared"));
    let second = Rc::clone(&first);
    assert_eq!(Rc::strong_count(&first), 2);
    assert!(Rc::ptr_eq(&first, &second)); // same allocation
    drop(second);
    assert_eq!(Rc::strong_count(&first), 1);
}
```

Cloning `Rc` or `Arc` clones the handle, not the underlying text. The inner value is dropped when the last strong owner is dropped. Strong-reference cycles can keep values alive indefinitely; use `Weak` for back-references when ownership should not be extended.

## Deref and Drop

`Deref` controls shared dereferencing and supports coercions such as `&String` to `&str`. `DerefMut` provides mutable dereferencing when exclusive access is valid. `Drop` implements cleanup when a value is destroyed.

`Arc<T>` does not make arbitrary T safe to mutate concurrently. `Arc<Mutex<T>>` combines shared ownership with synchronized access. `Rc<RefCell<T>>` is a common single-threaded combination.

Boxing is useful when indirection is required, but it is not automatically faster. It introduces allocation and pointer access. A recursive enum often needs indirection because its inline size would otherwise be infinite.

Related: [[Rust/Language/Advanced/interior-mutability|Interior mutability]] · [[Rust/Language/Data/memory|Drop and memory]].

Sources: [Smart pointers](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html), [Rc](https://doc.rust-lang.org/std/rc/), [Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html).
