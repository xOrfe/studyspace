---
title: Unsafe and FFI
tags: [rust, advanced, unsafe]
---

# Unsafe and FFI

`unsafe` marks operations whose memory-safety requirements the compiler cannot fully verify. It does not disable ordinary type checking or make undefined behavior acceptable.

**Complete example:**

```rust
fn main() {
    let value = 42_i32;
    let pointer: *const i32 = &value;
    // SAFETY: pointer comes from a live, aligned i32 reference;
    // the value remains initialized and is not modified here.
    let read = unsafe { *pointer };
    assert_eq!(read, 42);
}
```

Creating a raw pointer can be safe; dereferencing it requires establishing validity, alignment, initialization, and the relevant aliasing rules. “The address looks nonzero” is not a sufficient proof.

## Safe abstraction boundaries

An unsafe implementation can expose a safe interface only when every valid safe call preserves its invariants. Keep unsafe blocks small and put the reason each contract holds next to the operation. An `unsafe fn` imposes obligations on its caller; in edition 2024, use explicit unsafe blocks for unsafe operations within its body as well.

## Foreign function interface

FFI connects Rust to another language or ABI. A declaration does not verify the foreign implementation.

```rust
// Declaration snippet; requires a matching foreign library when used.
unsafe extern "C" {
    fn foreign_process(data: *const u8, len: usize) -> i32;
}
```

Check ABI, symbol availability, layout, ownership, pointer validity, and error conventions. `#[repr(C)]` provides C-compatible layout rules for applicable types; Rust's default struct layout is not a C ABI promise. Do not let a panic cross an ABI boundary that does not permit unwinding.

GPU raw pointers and hardware intrinsics introduce similar obligations, plus synchronization and device-specific constraints.

Related: [[Rust/GPU/cuda-oxide/safety|GPU safety contracts]].

Sources: [Unsafe Rust](https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html), [FFI](https://doc.rust-lang.org/nomicon/ffi.html).
