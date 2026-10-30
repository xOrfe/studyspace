---
title: GPU Programming with Rust
tags: [rust, gpu]
---

# GPU Programming with Rust

A GPU runs many pieces of work in parallel. Rust supplies types and ownership rules; a GPU programming system supplies device compilation, memory management, and launch APIs.

![Conceptual illustration: a CPU worker builds sequentially while GPU workers work on independent mosaic sections.](../assets/cpu-gpu.png)

*Conceptual analogy, not a hardware diagram. CPUs also execute parallel work, and GPUs have control flow and scheduling overhead. Parallelism helps only when the workload and data movement justify it.*

## Shared concepts

- [[Rust/GPU/foundations/host-device|Host, device, kernels, and transfers]]
- [[Rust/GPU/foundations/execution|Threads, blocks, grids, and warps]]
- [[Rust/GPU/foundations/memory|Memory hierarchy and access patterns]]
- [[Rust/GPU/foundations/streams|Streams, events, and synchronization]]
- [[Rust/GPU/foundations/operations|Vector addition, reduction, and matrix multiplication]]
- [[Rust/GPU/foundations/performance|Correctness and performance measurement]]

## Two programming models

| | cuda-oxide | cuTile Rust |
| --- | --- | --- |
| Main abstraction | Work performed by a thread | Work performed on a tile |
| Main control | Thread indices and cooperation | Tensor partitions and tile operations |
| Compilation | Rust to PTX ahead of time | Rust to Tile IR, JIT at launch |
| Notes | [[Rust/GPU/cuda-oxide/index|cuda-oxide notebook]] | [[Rust/GPU/cutile-rs/index|cuTile notebook]] |

[[Rust/GPU/comparison|SIMT vs tile comparison]] · [[Rust/GPU/glossary|Glossary]]

Ownership, references, slices, generics, and traits are useful background. Async Rust becomes relevant for asynchronous GPU pipelines.

## Source status

Checked **2026-10-30** against NVIDIA's live documentation and repository `main`. These are evolving research projects; a live page is not a frozen release. GPU examples here have **not been compiled or executed on GPU hardware**. See [[Rust/sources|Sources and verification]] for details.

Sources: [CUDA Rust](https://nvidia.github.io/cuda-rust/), [official repository](https://github.com/NVIDIA/cuda-rust).
