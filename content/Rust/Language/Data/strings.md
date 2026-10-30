---
title: String vs str
tags: [rust, strings]
---

# String vs str

`String` owns a growable UTF-8 buffer. `str` is a UTF-8 string slice type, usually accessed as `&str`. Use `String` to build or own text and `&str` to read borrowed text.

| Type | Owns text? | Growable? | Example |
| --- | --- | --- | --- |
| `String` | Yes | Yes | `String::from("hello")` |
| `&str` | No | No | `"hello"`, `text.as_str()` |
| `&mut str` | No | No | Restricted in-place edits preserving UTF-8 |

**Complete example:**

```rust
fn greet(name: &str) -> String {
    format!("Hello, {name}")
}

fn main() {
    let mut name = String::from("Ada");
    name.push('!');
    assert_eq!(greet(&name), "Hello, Ada!");
    assert_eq!(greet("Lin"), "Hello, Lin");
    assert_eq!("é".len(), 2); // bytes
    assert_eq!("é".chars().count(), 1); // Unicode scalar values
}
```

`&String` can coerce to `&str`, so a parameter of type `&str` accepts both borrowed strings and string literals. Prefer it over `&String` when ownership-specific operations are unnecessary.

## UTF-8 boundaries

Rust does not support `text[0]` as a character lookup. A UTF-8 character may occupy multiple bytes. A range such as `&text[0..2]` uses **byte offsets** and panics if either boundary splits a character. `text.get(0..2)` returns `Option<&str>` instead.

`.chars()` yields Unicode scalar values, not necessarily user-perceived characters: combined accents and emoji may contain several scalars. Grapheme handling needs a suitable Unicode library.

A string literal has type `&'static str`; a borrowed `&str` in general does not have to be static.

Related: [[Rust/Language/Data/arrays-vec-slices|Slices]] · [[Rust/Language/Data/lifetimes|Lifetimes]].

Sources: [Strings](https://doc.rust-lang.org/book/ch08-02-strings.html), [str](https://doc.rust-lang.org/std/primitive.str.html).
