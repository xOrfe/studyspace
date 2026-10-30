---
title: Structs and Methods
tags: [rust, modeling]
---

# Structs and Methods

A **struct** groups named fields into a type. An `impl` block defines behavior associated with that type. Methods receive `self`; associated functions do not.

**Complete example:**

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn square(side: u32) -> Self { // associated function
        Self { width: side, height: side }
    }

    fn area(&self) -> u32 { // borrows the rectangle
        self.width * self.height
    }

    fn widen(&mut self, extra: u32) {
        self.width += extra;
    }
}

fn main() {
    let mut shape = Rectangle::square(3);
    shape.widen(1);
    assert_eq!(shape.area(), 12);
}
```

| Receiver | Meaning |
| --- | --- |
| `&self` | Read through a shared borrow |
| `&mut self` | Borrow exclusively, usually to modify |
| `self` | Consume the value; ownership passes into the method |

`Self` means the type being implemented. Rust automatically borrows a receiver for common method calls, so `shape.area()` does not need an explicit `(&shape).area()`.

Tuple structs have positional fields: `struct Meters(f64);`. Unit structs have no fields: `struct Ready;`. A **newtype** such as `Meters` distinguishes a quantity from an otherwise identical `f64`.

Fields can be destructured: `let Rectangle { width, height } = shape;`. Moving a non-`Copy` field can partially move a struct, limiting later use of the whole value.

Struct fields and methods are private by default across module boundaries. Add `pub` deliberately to expose an interface.

Related: [[Rust/Language/Abstraction/modules|Visibility]] · [[Rust/Language/Abstraction/traits|Traits]].

Sources: [Structs](https://doc.rust-lang.org/book/ch05-01-defining-structs.html), [methods](https://doc.rust-lang.org/book/ch05-03-method-syntax.html).
