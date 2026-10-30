---
title: cuda-oxide Notebook
tags: [rust, gpu, cuda-oxide]
---

# cuda-oxide Notebook

cuda-oxide compiles device Rust into PTX for CUDA's **SIMT** execution model. Code describes the work of individual threads; indexing and cooperation remain visible to the programmer.

```mermaid
flowchart LR
    R["Rust kernel"] --> C["cuda-oxide compiler"]
    C --> P["PTX"]
    P --> D["CUDA driver compilation"]
    D --> G["GPU execution"]
```

## Notes

- [[Rust/GPU/cuda-oxide/setup|Environment and setup]]
- [[Rust/GPU/cuda-oxide/first-kernel|A vector-add kernel]]
- [[Rust/GPU/cuda-oxide/indexing|Indexing and bounds]]
- [[Rust/GPU/cuda-oxide/buffers-launches|Buffers and launches]]
- [[Rust/GPU/cuda-oxide/safety|Safety contracts]]
- [[Rust/GPU/cuda-oxide/shared-memory|Shared memory and synchronization]]
- [[Rust/GPU/cuda-oxide/reduction-matmul|Reduction and matrix multiplication]]
- [[Rust/GPU/cuda-oxide/async|Asynchronous device operations]]
- [[Rust/GPU/cuda-oxide/advanced-hardware|Warps, tensor cores, TMA, and clusters]]

Device Rust has target-specific capabilities and restrictions. CPU APIs such as ordinary OS threads, file I/O, and an arbitrary dependency tree should not be assumed to work inside a kernel.

The notes emphasize concepts and the documented checked-launch path. Unsafe APIs require the caller to establish their contracts.

**Source snapshot:** NVIDIA live cuda-oxide book checked 2026-10-30. The book's landing page still describes v0.1.0 alpha; release listings contain newer versions. These notes do not claim that live-book APIs belong to a particular release. GPU examples have not been compiled or run locally.

Sources: [cuda-oxide book](https://nvidia.github.io/cuda-rust/cuda-oxide/), [releases](https://github.com/NVIDIA/cuda-rust/releases).
