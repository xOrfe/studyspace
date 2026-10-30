---
title: A cuda-oxide Vector-Add Kernel
tags: [rust, gpu, cuda-oxide]
---

# A cuda-oxide Vector-Add Kernel

The kernel computes `c[i] = a[i] + b[i]`. Each thread receives its own output location. For the notebook's shared example, `[1,2,3,4] + [10,20,30,40]` produces `[11,22,33,44]`.

**Device-module snippet — requires cuda-oxide's host wrapper and build tooling; not a standalone program.** The syntax follows the live quick-start API checked 2026-10-30. Not compiled or executed locally.

```rust
use cuda_device::{kernel, launch_contract, thread, DisjointSlice};

#[kernel]
#[launch_contract(domain = 1, block = (256, 1, 1))]
pub fn add_vectors(
    a: &[f32],
    b: &[f32],
    mut output: DisjointSlice<f32>,
) {
    let index = thread::index_1d();
    let i = index.get();
    if let Some(slot) = output.get_mut(index) {
        *slot = a[i] + b[i];
    }
}
```

| Part | Role |
| --- | --- |
| `#[kernel]` | Device entry point |
| `launch_contract` | Declares a one-dimensional launch and exact block shape |
| `&[f32]` | Read-only inputs |
| `DisjointSlice` | Restricted parallel output access |
| `thread::index_1d()` | Creates an index witness for this thread |
| `get_mut(index)` | Returns a mutable slot only when in bounds |

The host must provide inputs covering every output index, a matching prepared launch, and valid buffers. Checking the output index alone does not establish that an arbitrarily shorter input has that index.

Source: [Official quick start](https://nvidia.github.io/cuda-rust/cuda-oxide/). See the linked source for the complete host program; [[Rust/GPU/cuda-oxide/buffers-launches|buffers and launches]] explains its parts.
