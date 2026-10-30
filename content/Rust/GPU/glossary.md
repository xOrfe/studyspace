---
title: GPU Glossary
tags: [rust, gpu, reference]
---

# GPU Glossary

| Term | Meaning |
| --- | --- |
| CUDA | NVIDIA's GPU computing platform and programming ecosystem |
| CUDA Rust | NVIDIA's Rust platform combining runtime crates and two kernel models |
| cuda-oxide | Rust-to-CUDA SIMT compiler and supporting tools |
| cuTile Rust / cutile-rs | Tile-based kernel programming in Rust |
| CUDA Tile IR | Intermediate representation for tile computations |
| CuTe | Layout and tensor abstractions associated with CUTLASS; distinct from cuTile |
| CUTLASS | NVIDIA's infrastructure for high-performance linear algebra kernels |
| PTX | Virtual instruction-set representation compiled for a target GPU |
| cubin | Compiled device binary for a GPU architecture |
| JIT | Compilation when code is needed, such as first launch |
| SIMT | Single Instruction, Multiple Threads |
| Tensor | Multidimensional data view; a mathematical concept, not a synonym for Tensor Core |
| Tile | A subregion used as a computation unit |
| Tensor Core | Specialized hardware for supported matrix operations |
| SM | Streaming multiprocessor |
| TMA | Tensor Memory Accelerator; hardware-assisted data movement on supported GPUs |
| Coalescing | Combining neighboring memory accesses into efficient transactions |
| Occupancy | Resident active warps relative to the hardware limit |
| cuda-bindings | Low-level CUDA driver bindings |
| cuda-core | Runtime abstractions for devices, streams, memory, and launches |
| cuda-async | Composable asynchronous device operations |

These names are related, but **cuTile is not CuTe**, and a Rust GPU library is not necessarily part of NVIDIA's CUDA Rust project.

Sources: [CUDA Rust repository](https://github.com/NVIDIA/cuda-rust), [CUTLASS documentation](https://docs.nvidia.com/cutlass/). Checked 2026-10-30.
