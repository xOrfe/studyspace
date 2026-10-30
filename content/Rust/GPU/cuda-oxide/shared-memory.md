---
title: Shared Memory and Synchronization
tags: [rust, gpu, cuda-oxide]
---

# Shared Memory and Synchronization

Shared memory is scratch storage accessible to threads in one block. It is useful when those threads reuse data or exchange intermediate results.

```text
global data
    │ cooperative load
    ▼
shared tile
    │ barrier: producers finished
    ▼
threads read shared values and compute
    │ barrier before overwriting reused storage
    ▼
next tile
```

**Conceptual pseudocode — not compilable Rust:**

```text
shared[local_id] = valid(global_id) ? input[global_id] : 0
block_barrier()
consume shared values
block_barrier()  // needed before reuse if readers might still be active
```

Out-of-range threads can supply a neutral value while still participating in required barriers. Returning early before a collective barrier can break the synchronization contract.

## Static and dynamic storage

Static shared storage has a size known at compilation. Dynamic shared storage receives its byte budget from the launch. In either case, all concurrent accesses must follow the API's aliasing and synchronization rules.

The cuda-oxide documentation exposes shared arrays and block synchronization, with unsafe obligations around shared accesses. Use its current examples for exact declarations rather than treating shared memory as an ordinary Rust mutable global.

## Performance considerations

Shared memory can reduce repeated global loads. However, it introduces barriers, consumes a finite per-block budget, and can suffer bank conflicts. Double buffering alternates storage regions so loading and computation can overlap; it also makes ownership of each stage and completion dependencies more complex.

A block barrier applies within one block, not across the entire grid. Multi-block cooperation usually needs separate launches or specialized supported mechanisms.

Related: [[Rust/GPU/cuda-oxide/reduction-matmul|Cooperative algorithms]].

Source: [Shared memory and synchronization](https://nvidia.github.io/cuda-rust/cuda-oxide/advanced/shared-memory-and-synchronization.html). Checked 2026-10-30.
