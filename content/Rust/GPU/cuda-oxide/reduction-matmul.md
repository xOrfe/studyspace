---
title: SIMT Reduction and Matrix Multiplication
tags: [rust, gpu, cuda-oxide, algorithms]
---

# SIMT Reduction and Matrix Multiplication

Vector addition assigns independent outputs. Reductions and tiled matrix multiplication add cooperation and reuse.

## Reduction

For `[1,2,3,4]`, one tree combines `1+2` and `3+4`, then combines `3+7` into `10`.

**Conceptual pseudocode — not a cuda-oxide API example:**

```text
each thread loads one value or zero into shared storage
barrier
for stride = block_width/2, ... , 1:
    if local_id < stride:
        shared[local_id] += shared[local_id + stride]
    barrier
thread 0 writes the block's partial sum
another launch reduces partial sums if several blocks were used
```

This simple form assumes a power-of-two block width. The bounds handling belongs in the load, and all required threads reach every barrier. A block barrier cannot combine independently scheduled blocks.

Warp shuffles can exchange register values without shared storage, but masks and participating lanes must satisfy the intrinsic's contract.

## Matrix multiplication

For the shared 2×2 example, `C = [[19,22],[43,50]]`. A basic SIMT mapping assigns one output cell to each thread and computes its dot product.

```text
thread for (row, col):
    total = 0
    for k in 0..K:
        total += A[row,k] * B[k,col]
    C[row,col] = total
```

A tiled implementation cooperatively loads chunks of A and B into shared memory. Threads reuse those chunks for many output operations, synchronize, and advance along K. Partial edge tiles need valid loads or padding.

Tensor-core implementations add layout, alignment, precision, and architecture constraints. A scalar dot-product kernel does not automatically use tensor cores.

Related: [[Rust/GPU/foundations/operations|CPU arithmetic reference]] · [[Rust/GPU/cutile-rs/reduction-matmul|Tile interpretation]].

Sources: [Warp programming](https://nvidia.github.io/cuda-rust/cuda-oxide/advanced/warp-level-programming.html), [shared memory](https://nvidia.github.io/cuda-rust/cuda-oxide/advanced/shared-memory-and-synchronization.html). Checked 2026-10-30; algorithms above are explanatory pseudocode.
