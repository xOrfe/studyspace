---
title: Modules, Crates, and Visibility
tags: [rust, abstraction]
---

# Modules, Crates, and Visibility

A **package** is managed by Cargo. A **crate** is a compilation unit, such as a library or executable. A **module** groups names inside a crate. These are related but distinct boundaries.

**Complete example:**

```rust
mod math {
    pub fn square(n: i32) -> i32 {
        multiply(n, n)
    }

    fn multiply(a: i32, b: i32) -> i32 { a * b }
}

use math::square;

fn main() {
    assert_eq!(square(4), 16);
}
```

`square` is public; `multiply` is an implementation detail. `use` brings a path into scope, rather than copying code or loading a file at runtime.

| Path or keyword | Meaning |
| --- | --- |
| `crate::...` | Start at this crate's root |
| `self::...` | Start at this module |
| `super::...` | Start at the parent module |
| `pub` | Public, subject to the visibility of enclosing modules |
| `pub(crate)` | Visible within the current crate |
| `pub use ...` | Re-export a name through this module |

Private items are accessible within their defining module and its descendants. A public struct's fields remain private unless individually marked `pub`; enum variants share the enum's visibility.

## File organization

`mod math;` in `src/lib.rs` declares a module whose body can live in `src/math.rs`. Nested modules can use `src/math/vector.rs` with `mod vector;` inside the math module. A filename alone does not declare a module.

An external dependency is declared in `Cargo.toml`; importing it with `use` only works after Cargo makes the crate available.

Related: [[Rust/Language/Foundations/cargo|Cargo]] · [[Rust/Language/Advanced/testing|Tests]].

Source: [Managing growing projects](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html).
