---
title: Streams, Events, and Synchronization
tags: [rust, gpu, async]
---

# Streams, Events, and Synchronization

A **stream** is an ordered queue of device operations. An **event** marks progress in a stream and can support waiting, dependencies, or elapsed-time measurement.

```text
Stream A: upload A ──► kernel A ──► event ready
                                      │ dependency
Stream B: unrelated work ──────────────▼──► consume A
Host:     submit work ──► other work ──► wait/read result
```

Operations in one stream respect ordering. Separate streams can overlap when dependencies, hardware resources, and runtime behavior allow it. Two streams do not guarantee simultaneous execution.

## Synchronization levels

| Mechanism | Coordinates |
| --- | --- |
| Block barrier | Cooperating threads inside one block |
| Event dependency | Device work across streams |
| Stream synchronization | Host waits for a stream's submitted work |
| Device synchronization | Host waits broadly for device work |

These are not interchangeable. A host-side stream wait cannot repair a data race inside a kernel. A block barrier cannot ensure another block has completed.

## Buffer lifetime

A Rust launch call may return while the GPU still reads its inputs. The runtime must retain ownership or borrows until completion. Reusing an output buffer too early can conflict with in-flight work even when the CPU has finished submitting the launch.

Pipelines can overlap upload of chunk B with computation on chunk A and download of chunk C. This needs independent buffers, explicit dependencies, and appropriate transfer support. A synchronization after every step removes most overlap.

GPU asynchronous execution and Rust `async` are related layers: a GPU stream can execute asynchronously even when the host API is not an `async fn`.

Related: [[Rust/Language/Advanced/async|Rust async]] · [[Rust/GPU/cuda-oxide/async|Device operations]].

Source: [Scheduling and streams](https://nvidia.github.io/cuda-rust/cuda-oxide/async-programming/scheduling-and-streams.html).
