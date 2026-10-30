---
title: Generics and Traits
tags: [rust, abstraction]
---

# Generics and Traits

**Generics** parameterize code over types or constants. A **trait** describes behavior a type can provide. Trait bounds state which operations generic code may use.

**Complete example:**

```rust
trait Area {
    fn area(&self) -> f64;
}

struct Square { side: f64 }

impl Area for Square {
    fn area(&self) -> f64 { self.side * self.side }
}

fn twice<T: Area>(shape: &T) -> f64 {
    shape.area() * 2.0
}

fn main() {
    assert_eq!(twice(&Square { side: 3.0 }), 18.0);
}
```

`T: Area` means “T implements Area.” The compiler checks the body using that contract and generally generates specialized code for concrete types (monomorphization).

Bounds can use a `where` clause for readability:

```rust
// Snippet
fn inspect<T>(item: &T)
where
    T: std::fmt::Debug + Clone,
{
    println!("{item:?}");
}
```

Traits can include default methods, associated types, and constants. `#[derive(Debug, Clone)]` can generate common trait implementations when the fields support them.

## Const generics

`fn size<T, const N: usize>(_: &[T; N]) -> usize { N }` works for arrays of different lengths while preserving each length in its type. Const generics are useful for dimensions and fixed-size buffers; not every arbitrary compile-time expression is accepted in every context.

Rust's coherence rules generally require an implementation to involve a local trait or local type. A local newtype can adapt behavior for an external type.

Related: [[Rust/Language/Abstraction/dispatch|Associated types and dispatch]].

Sources: [Generics](https://doc.rust-lang.org/book/ch10-01-syntax.html), [traits](https://doc.rust-lang.org/book/ch10-02-traits.html).
