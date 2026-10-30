---
title: GPU Memory Hierarchy
tags: [rust, gpu, memory]
---

# GPU Memory Hierarchy

GPU memory spaces differ in scope, capacity, latency, and management. “Local” describes visibility, not necessarily physical proximity.

| Storage | Visibility | Role |
| --- | --- | --- |
| Registers | One thread | Fast scalar working values; limited capacity |
| Local memory | One thread | Per-thread storage that may reside in device memory, including spills |
| Shared memory | One block | Explicitly managed scratch space for cooperation |
| Global memory | Device-accessible | Large input/output buffers |
| Caches | Hardware-managed | Reuse recently accessed device data |

```mermaid
flowchart LR
    G["global input"] --> S["shared tile: reusable within block"]
    S --> R["register values: thread computation"]
    R --> O["global output"]
```

The diagram is a common algorithmic data flow, not a requirement for every kernel.

## Coalescing

When neighboring warp lanes access neighboring addresses, hardware can often serve them with fewer memory transactions. Scattered addresses may require more transactions for the same useful bytes.

For row-major matrices, neighboring columns are contiguous. Walking a column uses a stride equal to the row width, which can make a naive access pattern less efficient.

## Shared-memory tiling

Load a region once, reuse it for several computations, then store results. The extra coordination pays off only when it saves enough global traffic or enables efficient computation.

Shared memory needs correct barriers before another thread consumes written data and before storage is reused. Bank conflicts can serialize certain access patterns. More shared memory or registers per block can also reduce the number of simultaneously resident blocks.

## Host memory

Pinned host memory can support efficient asynchronous transfers. Unified/managed memory and mapped memory have different access and migration rules; they do not eliminate the need to reason about where data resides and when it is valid.

Related: [[Rust/GPU/cuda-oxide/shared-memory|Shared memory and synchronization]] · [[Rust/GPU/foundations/performance|Performance]].

Sources: [CUDA Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/), [memory and data movement](https://nvidia.github.io/cuda-rust/cuda-oxide/gpu-programming/memory-and-data-movement.html).
