---
title: Primitive Types and Conversions
tags: [rust, foundations]
---

# Primitive Types and Conversions

Rust is statically typed: each value's type is known at compile time, often through inference. An annotation such as `let n: u32 = 3;` resolves ambiguity or documents intent.

| Family | Types | Meaning |
| --- | --- | --- |
| Signed integers | `i8` … `i128`, `isize` | Positive and negative whole numbers |
| Unsigned integers | `u8` … `u128`, `usize` | Nonnegative whole numbers |
| Floating point | `f32`, `f64` | Approximate real-number arithmetic |
| Boolean | `bool` | `true` or `false` |
| Character | `char` | One Unicode scalar value; four bytes |
| Unit | `()` | No meaningful value |

`usize` is the pointer-sized unsigned type used for indexing. Its width depends on the target. Unsuffixed integer and float literals usually default to `i32` and `f64` when context does not choose another type.

## Conversions

**Complete example:**

```rust
fn main() {
    let small: u8 = 200;
    let wide = u32::from(small); // lossless
    let too_large = u8::try_from(300_u16); // checked, returns Result
    let parsed = "42".parse::<i32>(); // text to number
    assert_eq!(wide, 200);
    assert!(too_large.is_err());
    assert_eq!(parsed, Ok(42));
    assert_eq!(5 / 2, 2); // integer division
}
```

`as` performs a cast, not a general validation step. Narrowing an integer with `as` can discard high bits. Prefer `From` for infallible conversions and `TryFrom` when a value may not fit.

Ordinary integer overflow can panic when overflow checks are enabled; it is usually wrapping in release profiles. Use `checked_add`, `saturating_add`, or `wrapping_add` when overflow behavior is part of the algorithm.

Floating-point values are approximate: `0.1 + 0.2` need not equal `0.3`. Compare with a tolerance appropriate to the problem, particularly in GPU results.

Related: [[Rust/Language/Modeling/result|Result]] · [[Rust/Language/Data/strings|Strings and Unicode]].

Sources: [Data types](https://doc.rust-lang.org/book/ch03-02-data-types.html), [conversions](https://doc.rust-lang.org/rust-by-example/conversion.html).
