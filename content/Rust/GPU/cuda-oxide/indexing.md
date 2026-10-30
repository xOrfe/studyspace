---
title: cuda-oxide Indexing and Bounds
tags: [rust, gpu, cuda-oxide]
---

# cuda-oxide Indexing and Bounds

An index maps a thread to the data it processes. A correct mapping must cover the intended elements without creating conflicting writes.

For a one-dimensional output:

```text
i = block_x * block_width + local_thread_x

N = 10, block_width = 4
block 0 → i 0..3
block 1 → i 4..7
block 2 → i 8..11; only 8 and 9 belong to the output
```

cuda-oxide's `ThreadIndex` is more than an integer: it is a witness used by `DisjointSlice` to restrict mutable access. `index.get()` exposes a number for reading inputs, while the witness is passed into `get_mut` for output access.

## Two dimensions

For a row-major matrix with `width` columns:

```text
flat_index = row * width + column
valid = row < height AND column < width
```

The row stride is the distance between row starts; it is not always equal to the visible width for a padded or strided view. Mixing widths produces incorrect addressing and may create collisions in an unchecked implementation.

The live safety documentation describes typed index spaces for fixed row strides and host-bound row-width mechanisms for runtime dimensions. Use the matching API for the selected checkout; do not manufacture a raw integer and assume it is equivalent to a witness.

## Boundary behavior

An overprovisioned launch is normal when the element count is not divisible by the block width. Guard every access that can exceed its valid range. In a cooperative kernel, early return before a required block barrier is a separate hazard: all required threads must participate.

Related: [[Rust/GPU/cuda-oxide/safety|Safety]] · [[Rust/GPU/cuda-oxide/shared-memory|Barriers]].

Sources: [Safety model](https://nvidia.github.io/cuda-rust/cuda-oxide/gpu-safety/the-safety-model.html), [execution model](https://nvidia.github.io/cuda-rust/cuda-oxide/gpu-programming/execution-model.html). Checked 2026-10-30; live APIs.
