---
title: Broadcasting and Boundary Tiles
tags: [rust, gpu, cutile]
---

# Broadcasting and Boundary Tiles

**Broadcasting** expresses an operation between values of compatible shapes without requiring the programmer to create repeated copies manually.

For a scalar multiplier:

```text
scalar: 2       input tile: [1, 2, 3]
broadcast: [2, 2, 2]
result:    [2, 4, 6]
```

The expanded row shows the mathematical meaning, not a required memory allocation.

The official SAXPY example computes `y = a*x + y`:

```rust
// Kernel-body snippet from the documented operation pattern.
let scale = a.broadcast(y.shape());
let input = x.load_like(y);
let previous = y.load();
y.store(scale * input + previous);
```

Here `a` is scalar, `x` is a shared input tensor, and `y` is the mutable output view. This snippet needs the enclosing cuTile kernel signature. It has not been compiled or executed locally.

## Boundary tiles

For ten elements and a logical tile width of four:

```text
tile 0: [0 1 2 3]
tile 1: [4 5 6 7]
tile 2: [8 9 - -]  last two positions are outside the data
```

Handling those positions is part of the load/store contract. Do not assume every API silently pads with zero. A safe interface may reject an unsupported shape, check bounds, or provide an explicit boundary policy. Consult the selected operation's documentation.

For conceptual reduction padding, zero is neutral for addition; multiplication uses one, and maximum requires an appropriate minimum or negative-infinity value. Output stores must still exclude invalid positions.

For multidimensional broadcasting, align compatible axes according to the API's rules. A row vector and column vector represent different shapes even when their element counts match.

Related: [[Rust/GPU/cutile-rs/reduction-matmul|Reductions]] · [[Rust/GPU/foundations/performance|Correctness]].

Source: [Official SAXPY example](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/cutile-examples/examples/saxpy.rs). Checked 2026-10-30; boundary discussion is conceptual, not a claim of automatic padding.
