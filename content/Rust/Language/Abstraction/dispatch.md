---
title: Associated Types and Dispatch
tags: [rust, abstraction]
---

# Associated Types and Dispatch

An **associated type** is a type chosen by a trait implementation. `Iterator::Item` describes what a particular iterator produces.

**Complete example:**

```rust
fn sum_values<I>(items: I) -> i32
where
    I: Iterator<Item = i32>,
{
    items.sum()
}

fn main() {
    assert_eq!(sum_values([1, 2, 3].into_iter()), 6);
}
```

`Item = i32` constrains the associated type. A generic parameter lets callers select a type parameter; an associated type is fixed by the selected implementation.

## Static vs dynamic dispatch

| Form | Selection | Typical benefit |
| --- | --- | --- |
| `fn draw<T: Draw>(x: &T)` | Concrete type known at compile time | Specialization and inlining opportunities |
| `fn draw(x: &impl Draw)` | Generic argument shorthand | Concise static interface |
| `fn draw(x: &dyn Draw)` | Method selected through a vtable | Runtime polymorphism |

A trait object such as `&dyn Draw` combines a data pointer and dispatch metadata. Only dyn-compatible traits can form trait objects; methods with unconstrained generic type parameters commonly prevent this.

`Box<dyn Draw>` owns a dynamically selected value, while `&dyn Draw` borrows it. Dynamic dispatch itself does not require a heap allocation: the reference form can point to an ordinary local value.

## Returning `impl Trait`

`fn values() -> impl Iterator<Item = i32> { 0..3 }` hides one concrete return type while exposing its capabilities. Different branches cannot return unrelated concrete iterator types merely because both implement `Iterator`; use a common representation or an appropriate trait object.

Related: [[Rust/Language/Abstraction/traits|Traits]] · [[Rust/Language/Advanced/smart-pointers|Box]].

Sources: [Advanced traits](https://doc.rust-lang.org/book/ch20-02-advanced-traits.html), [trait objects](https://doc.rust-lang.org/book/ch18-02-trait-objects.html).
