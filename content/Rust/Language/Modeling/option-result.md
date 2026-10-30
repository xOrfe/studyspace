---
title: Option vs Result
tags: [rust, comparisons]
---

# Option vs Result

Both types represent alternatives, but they communicate different meanings.

| | `Option<T>` | `Result<T, E>` |
| --- | --- | --- |
| Question | Is a value present? | Did the operation succeed? |
| Positive case | `Some(T)` | `Ok(T)` |
| Other case | `None` | `Err(E)` |
| Extra explanation | No | Error value `E` |
| Typical example | First item of an empty list | Parsing malformed text |

**Complete example:**

```rust
fn main() {
    let empty: Vec<i32> = vec![];
    let first = empty.first(); // Option<&i32>
    let parsed = "bad".parse::<i32>(); // Result<i32, ParseIntError>
    assert!(first.is_none());
    assert!(parsed.is_err());

    let required = first.ok_or("list must not be empty");
    assert_eq!(required, Err("list must not be empty"));
}
```

`.ok_or(error)` converts absence into a meaningful error. `.ok()` converts `Result<T, E>` into `Option<T>` and **discards the error**. Use it only when that information truly is irrelevant.

Nested types can express more detail. `Result<Option<T>, E>` means the operation can fail, succeed with a value, or succeed with no value. For example, a database lookup may fail to connect or successfully find no record.

`?` propagates the surrounding abstraction: `None` in an Option-returning function, or an error in a Result-returning function. It does not automatically turn `None` into an arbitrary error; use `.ok_or(...)` to define that meaning.

Related: [[Rust/Language/Modeling/option|Option]] · [[Rust/Language/Modeling/result|Result]].

Sources: [Option](https://doc.rust-lang.org/std/option/), [Result](https://doc.rust-lang.org/std/result/).
