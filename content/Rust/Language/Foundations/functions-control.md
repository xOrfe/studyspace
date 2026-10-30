---
title: Functions, Expressions, and Control Flow
tags: [rust, foundations]
---

# Functions, Expressions, and Control Flow

A function declares parameter types and an optional return type. Rust blocks are expressions: the final expression without a semicolon becomes the block's value.

**Complete example:**

```rust
fn doubled(n: i32) -> i32 {
    n * 2 // no semicolon: return this value
}

fn main() {
    let n = if true { doubled(3) } else { 0 };
    let mut sum = 0;
    for value in 1..=n { // inclusive range
        if value % 2 == 0 {
            sum += value;
        }
    }
    assert_eq!(sum, 12); // 2 + 4 + 6
}
```

`let n = ...;` is a statement. `n * 2` is an expression. Adding a semicolon changes the function body above to produce `()`, which does not match `-> i32`. `return value;` explicitly exits early.

| Form | Purpose |
| --- | --- |
| `if condition { a } else { b }` | Choose a value; branches must have compatible types |
| `for item in iterable` | Visit items |
| `while condition` | Repeat while a Boolean condition holds |
| `loop` | Repeat until `break` |
| `break value` | End a `loop` expression with a value |
| `continue` | Start the next iteration |

`1..4` includes 1, 2, 3. `1..=4` also includes 4. Conditions must be `bool`; integers are not implicitly truthy.

**Snippet:**

```rust
let mut attempt = 0;
let answer = loop {
    attempt += 1;
    if attempt == 3 { break attempt * 10; }
};
assert_eq!(answer, 30);
```

`match` is another expression, designed for exhaustive branching over patterns.

Related: [[Rust/Language/Modeling/enums|Enums and pattern matching]].

Sources: [Functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html), [control flow](https://doc.rust-lang.org/book/ch03-05-control-flow.html).
