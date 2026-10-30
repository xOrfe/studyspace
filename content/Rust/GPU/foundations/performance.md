---
title: Correctness and Performance Measurement
tags: [rust, gpu, performance]
---

# Correctness and Performance Measurement

GPU speed depends on arithmetic, memory traffic, launch overhead, transfers, and compilation. Measure the boundary that corresponds to the question.

| Measurement | Includes |
| --- | --- |
| Kernel time | GPU execution between correctly ordered events |
| End-to-end time | Transfers, launches, waits, and host work |
| First-call latency | May also include initialization and JIT compilation |
| Steady-state throughput | Repeated work after warmup |

Timing only a launch call usually measures submission rather than completion. Use device events for device elapsed time, and host timers with a completion wait for end-to-end measurements. Separate cuTile's first-launch compilation from repeated execution.

## Correctness first

Compare against a small CPU implementation, including non-multiple sizes, empty inputs where supported, and unusual dimensions. Validate shape and indexing before tuning. Use a suitable absolute/relative tolerance for floating-point output rather than assuming bitwise equality across different reduction orders.

## Reading the bottleneck

**Vector addition** does little arithmetic per byte transferred and is often bandwidth-bound. **Matrix multiplication** can reuse each input value many times, increasing arithmetic intensity. Small problems may be dominated by fixed overhead in either case.

For f32 vector addition, a simple traffic estimate is two 4-byte loads plus one 4-byte store per output: roughly `12 × N` useful bytes, excluding caching and other effects.

## Profilers

Nsight Systems shows host/device timelines and overlap. Nsight Compute investigates individual kernels: memory access, occupancy, register pressure, and execution behavior. High occupancy alone does not prove high performance.

Tile size, block size, data layout, and numerical precision are tuning choices. Change them with measurements and recheck correctness. Comparisons should report hardware, toolchain, dimensions, timing boundary, and warmup behavior.

Sources: [CUDA Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/), [cuTile profiling guide](https://nvidia.github.io/cuda-rust/cutile/latest/guide/debugging-and-profiling.html).
