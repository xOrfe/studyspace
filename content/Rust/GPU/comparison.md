---
title: SIMT vs Tile Programming
tags: [rust, gpu, comparisons]
---

# SIMT vs Tile Programming

Both models describe parallel computation. They expose different units of work to the programmer.

| Concern | cuda-oxide / SIMT | cuTile / tile |
| --- | --- | --- |
| Addition | A thread computes selected elements | A program computes a tile |
| Addressing | Explicit thread/block indexing | Tensor shapes and partitions |
| Cooperation | Programmer handles shared memory and synchronization | Compiler handles normal tile-operation mapping |
| Compilation | Device Rust becomes PTX ahead of time | Kernel becomes Tile IR and is JIT-compiled |
| Control | Detailed thread and hardware control | Higher-level tensor operations |

For the same four-element addition:

```text
SIMT: thread 0 → C[0]; thread 1 → C[1]; …
Tile: tile 0 → C[0..2]; tile 1 → C[2..4]
Both: C = [11, 22, 33, 44]
```

The two-element tile is a teaching example, not a recommended performance configuration.

SIMT fits explicit CUDA-style coordination and irregular algorithms. Tile operations fit dense numerical work that can be described as loading regions, computing on them, and storing regions. Neither model guarantees a speedup for a particular workload.

Both share runtime components in CUDA Rust and can interoperate over compatible buffers and streams. That does not make a kernel's attributes or device types interchangeable.

High-level tile safety applies within its documented contracts. Raw operations and unsafe escape hatches require separate reasoning; Rust syntax alone is not proof of race freedom.

Related: [[Rust/GPU/cuda-oxide/index|cuda-oxide]] · [[Rust/GPU/cutile-rs/index|cuTile]].

Source: [NVIDIA: picking a model](https://nvidia.github.io/cuda-rust/#picking-a-model). Checked 2026-10-30; live documentation.
