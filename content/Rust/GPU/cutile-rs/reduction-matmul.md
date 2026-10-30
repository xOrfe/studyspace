---
title: Tile Reduction and Matrix Multiplication
tags: [rust, gpu, cutile, algorithms]
---

# Tile Reduction and Matrix Multiplication

## Reduction

A reduction combines values along an axis. A tile containing `[1,2,3,4]` reduces to `10` under addition. With multiple tiles, local partial sums must be combined in another stage or through a supported global-reduction design.

**Conceptual pseudocode — not cuTile API syntax:**

```text
tile = load input region
partial = reduce_sum(tile, selected_axis)
store partial in this region's unique output slot
combine partials in a subsequent operation if necessary
```

Reducing one tile does not automatically reduce an arbitrarily large tensor. The reduction axis determines the output shape. Floating-point results depend on the combination order.

## Matrix multiplication

For A shaped `[M,K]` and B shaped `[K,N]`, C has shape `[M,N]`. A program instance owns an output tile `[BM,BN]` and accumulates contributions along K.

```text
C tile: [BM, BN]
for each K region:
    A tile: [BM, BK]
    B tile: [BK, BN]
    accumulator += A tile × B tile
store accumulator
```

The notebook's 2×2 example gives `[[19,22],[43,50]]`, exactly as in the CPU and SIMT explanations.

The official GEMM example partitions input views and applies an `mma` operation to tile values. Its illustrated K loop uses `K / BK`, so copying that pattern requires respecting its divisibility assumption. An accumulator must also have the intended initial value; loading C is appropriate for accumulation only when C has been initialized accordingly.

**GEMM** commonly means `C = alpha*A*B + beta*C`, which is more general than a plain matrix product. Precision and accumulation type matter, particularly for tensor-core implementations.

Related: [[Rust/GPU/foundations/operations|Numerical examples]] · [[Rust/GPU/cuda-oxide/reduction-matmul|SIMT mapping]].

Source: [Official cuTile GEMM example](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/cutile-examples/examples/gemm.rs). Checked 2026-10-30; algorithms above are pseudocode, not executed GPU examples.
