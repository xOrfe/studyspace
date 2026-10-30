---
title: Testing and Documentation
tags: [rust, advanced]
---

# Testing and Documentation

Rust has built-in support for unit tests and executable documentation examples. Cargo coordinates compilation and execution.

**Complete library example — save as `src/lib.rs`:**

```rust
/// Doubles a small integer.
///
/// ```
/// let n = 3 * 2;
/// assert_eq!(n, 6);
/// ```
pub fn doubled(n: i32) -> i32 { n * 2 }

#[cfg(test)]
mod tests {
    use super::doubled;

    #[test]
    fn doubles_three() {
        assert_eq!(doubled(3), 6);
    }
}
```

`#[test]` marks a test function. `#[cfg(test)]` includes the test module for test builds. `assert_eq!` compares values and reports a mismatch. Unit tests can access private items through the module structure.

| Location or command | Meaning |
| --- | --- |
| Tests beside implementation | Unit tests |
| `tests/*.rs` | Integration tests using the public crate interface |
| Fenced examples in `///` comments | Documentation tests |
| `cargo test` | Run tests, including eligible doc examples |
| `cargo doc --open` | Generate and open API documentation |

The documentation example above is self-contained. In real API docs, call the documented function through its crate name so the example checks the interface itself.

Documentation should state units, ownership expectations, error cases, and panic conditions. Unsafe APIs must state the safety conditions callers must uphold. GPU APIs should additionally document synchronization, buffer lifetime, dimensions, and architecture requirements.

These are notes about Rust's testing tools, not exercises attached to the notebook.

Sources: [Writing tests](https://doc.rust-lang.org/book/ch11-01-writing-tests.html), [documentation tests](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html).
