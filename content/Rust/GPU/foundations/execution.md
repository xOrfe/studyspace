---
title: Threads, Blocks, Grids, and Warps
tags: [rust, gpu, foundations]
---

# Threads, Blocks, Grids, and Warps

In the SIMT model, you write the behavior of one logical **thread**. Threads form **blocks**, and a kernel launch creates a **grid** of blocks.

```text
Grid: 2 blocks, 4 threads per block

Block 0                         Block 1
local:   0   1   2   3          local:   0   1   2   3
global:  0   1   2   3          global:  4   5   6   7
```

For a one-dimensional launch:

```text
global_index = block_index * threads_per_block + thread_index
blocks = ceil(number_of_elements / threads_per_block)
```

With 10 elements and blocks of 4 threads, the launch contains 12 threads. Threads 10 and 11 must not access nonexistent elements.

## Cooperation boundaries

| Group | Meaning |
| --- | --- |
| Thread | One logical execution instance |
| Warp | NVIDIA execution group, normally 32 lanes |
| Block | Threads that can cooperate through block shared memory and barriers |
| Grid | All blocks in one launch |
| SM | Hardware multiprocessor that schedules resident blocks/warps |

A four-thread block in the drawing is intentionally small for readability; it does not mean a warp has four lanes.

Blocks can execute in any order. Ordinary kernels must not assume block 0 finishes before block 1. A block barrier does not synchronize the whole grid.

## Divergence

When lanes in a warp choose different paths, execution may serialize those paths under masks. Branching is legal, but the amount and pattern of divergence can affect efficiency. A logical thread is not an operating-system thread.

cuTile lets the compiler map tile operations onto this hardware hierarchy; its normal high-level code does not manually assign one scalar operation to each thread.

Related: [[Rust/GPU/cuda-oxide/indexing|Indexing]] · [[Rust/GPU/cutile-rs/tensors-tiles|Tiles]].

Source: [CUDA execution model](https://nvidia.github.io/cuda-rust/cuda-oxide/gpu-programming/execution-model.html).
