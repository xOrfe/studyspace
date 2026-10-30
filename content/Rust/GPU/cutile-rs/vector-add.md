---
title: cuTile Vector Addition
tags: [rust, gpu, cutile]
---

# cuTile Vector Addition

The arithmetic is still `output[i] = left[i] + right[i]`. cuTile expresses the operation over corresponding tiles instead of spelling out each thread index.

**Kernel-module snippet — follows the official `main` quick-start pattern checked 2026-10-30. Requires the cuTile crate, host code, and compatible CUDA environment. Not compiled or run locally.**

```rust
#[cutile::module]
mod vector_kernels {
    use cutile::core::*;

    #[cutile::entry()]
    fn add<const B: i32>(
        output: &mut Tensor<f32, { [B] }>,
        left: &Tensor<f32, { [-1] }>,
        right: &Tensor<f32, { [-1] }>,
    ) {
        let left_values = left.load_like(output);
        let right_values = right.load_like(output);
        output.store(left_values + right_values);
    }
}
```

`load_like` selects input values corresponding to the output region. Addition acts elementwise on the tile values; `store` writes the tile result.

For the notebook's small vectors, a conceptual tile width of two gives `[11,22]` and `[33,44]` in separate output regions. This is an arithmetic illustration, not a validated tuning choice.

## Host execution

The official quick start constructs two 1024-element tensors of ones and a zeroed output partitioned into 128-element regions. The generated launcher builds an operation; `.sync()` executes it. The launcher returns its runtime arguments in parameter order. Unpartitioning and copying to a host vector yields 1024 twos.

Partitioning creates exclusive output regions; it is not a CPU loop that launches a separate operating-system thread for each chunk. The first execution may include JIT compilation.

Related: [[Rust/GPU/cutile-rs/tensors-tiles|Shapes and partitions]] · [[Rust/GPU/cuda-oxide/first-kernel|SIMT version]].

Source: [Official cuTile quick start](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/README.md#quick-start). API snapshot: live `main`.
