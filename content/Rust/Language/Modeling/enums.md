---
title: Enums and Pattern Matching
tags: [rust, modeling]
---

# Enums and Pattern Matching

An **enum** describes a value that is one of several variants. Each variant may carry different data. This lets the type represent valid states directly.

**Complete example:**

```rust
enum Job {
    Waiting,
    Running { percent: u8 },
    Done(String),
}

fn describe(job: &Job) -> String {
    match job {
        Job::Waiting => "waiting".into(),
        Job::Running { percent } => format!("{percent}%"),
        Job::Done(message) => message.clone(),
    }
}

fn main() {
    assert_eq!(describe(&Job::Running { percent: 25 }), "25%");
    assert_eq!(describe(&Job::Waiting), "waiting");
    assert_eq!(describe(&Job::Done("ready".into())), "ready");
}
```

`match` is exhaustive: every possible variant must be covered. It is an expression, so the arms above produce a `String`. `_` matches anything remaining but can conceal a newly added variant that should receive special treatment.

Matching a borrowed enum borrows its contents through match ergonomics. Matching an owned enum can move owned payloads out of it.

## Shorter forms

**Snippet:**

```rust
let value = Some(7);
if let Some(n) = value {
    println!("{n}"); // handles one pattern
}

let Some(n) = value else {
    return; // let-else requires this branch to diverge
};
assert_eq!(n, 7);
```

Patterns also support tuples, structs, ranges, alternatives (`A | B`), and guards (`pattern if condition`). `if let` is convenient when only one case matters; `match` makes a full case distinction explicit.

Related: [[Rust/Language/Modeling/option|Option]] · [[Rust/Language/Modeling/result|Result]].

Sources: [Enums](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html), [patterns](https://doc.rust-lang.org/book/ch19-03-pattern-syntax.html).
