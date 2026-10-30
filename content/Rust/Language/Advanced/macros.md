---
title: Macros
tags: [rust, advanced]
---

# Macros

A macro generates code during compilation. Unlike a function, it can accept syntax and produce new syntax, including declarations. A macro is useful for repetitive structure or capabilities that normal function calls cannot express.

| Kind | Example | Purpose |
| --- | --- | --- |
| Declarative | `macro_rules!`, `vec!` | Pattern-based code generation |
| Derive procedural | `#[derive(Debug)]` | Generate implementations for a type |
| Attribute procedural | `#[some_attribute]` | Transform an annotated item |
| Function-like procedural | `some_macro!(...)` | Process a token stream |

**Complete example:**

```rust
macro_rules! pair {
    ($value:expr) => {{
        let value = $value; // evaluate the expression once
        (value, value)
    }};
}

fn main() {
    assert_eq!(pair!(2 + 3), (5, 5));
}
```

`$value:expr` matches an expression. The expansion above uses the local value twice, so it is suitable for a Copy value such as an integer; it is not a general duplication operation for owned resources.

Macro expansion still produces Rust code that must satisfy type and borrow checking. A macro is not a runtime interpreter. Procedural macros are implemented in a dedicated `proc-macro` crate and operate on tokens at compile time.

## GPU connection

cuTile's module and entry attributes identify kernel code for its compilation pipeline. cuda-oxide also uses attributes for modules, kernels, and launch contracts. Similar-looking attributes do not imply identical execution models or interchangeable APIs.

Prefer a function when ordinary parameters and return values express the operation clearly. Macros can make compiler diagnostics and generated behavior harder to inspect.

Sources: [Macros](https://doc.rust-lang.org/book/ch20-05-macros.html), [Reference: macros](https://doc.rust-lang.org/reference/macros.html).
