---
title: cuda-oxide Safety Contracts
tags: [rust, gpu, cuda-oxide, unsafe]
---

# cuda-oxide Safety Contracts

GPU safety includes valid addresses, exclusive writes, synchronization, and keeping resources alive until asynchronous work completes. Memory safety and mathematical correctness are separate requirements.

The live book presents three levels:

| Level | Typical mechanism | Responsibility |
| --- | --- | --- |
| Checked basic kernels | Shared inputs, disjoint output views, prepared launch | Use the verified contracts |
| Cooperative kernels | Shared memory and scoped unsafe operations | Prove synchronization and access rules |
| Raw hardware control | Architecture-specific intrinsics | Establish detailed hardware contracts |

`DisjointSlice` and trusted thread-index witnesses restrict which output element a thread may mutably access. A matching launch is part of that reasoning; arbitrary geometry is not interchangeable with checked preparation.

## What still needs reasoning

- An input must contain the elements the algorithm reads.
- Every mathematical output must actually be computed.
- Different streams need dependencies when sharing data.
- Shared-memory writes must become visible before consumers read them.
- Unsafe operations must satisfy their documented requirements.

For example, choosing a wrong but in-range matrix stride may produce wrong values without any out-of-bounds access. A type-safe launch is not a proof of the numerical algorithm.

A safety comment should explain why the required conditions hold at that operation. “This is GPU code” or “it worked once” is not a proof.

Related: [[Rust/Language/Advanced/unsafe-ffi|Unsafe Rust]] · [[Rust/GPU/cuda-oxide/indexing|Index spaces]].

Source: [Official safety model](https://nvidia.github.io/cuda-rust/cuda-oxide/gpu-safety/the-safety-model.html). Checked 2026-10-30; live documentation.
