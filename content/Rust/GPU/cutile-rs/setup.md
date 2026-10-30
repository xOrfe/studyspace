---
title: cuTile Environment and Setup
tags: [rust, gpu, cutile, setup]
---

# cuTile Environment and Setup

This is an environment reference, not an installation performed by the notebook.

The official `main` README checked **2026-10-30** specifies stable Rust 1.89+ and Linux, with Ubuntu 24.04 tested. CUDA 13.3 is recommended there. GPU support and required Tile IR versions vary by architecture and operation; GPUs below `sm_80` are listed as unsupported.

Do not interpret those baselines as support for every feature. Newer low-precision and raw operations can require a newer toolkit and a particular GPU. The release list also describes cuTile 0.4.0 as built on Tile IR 13.4, so match instructions to the chosen release or checkout.

## Toolkit selection

`CUDA_TOOLKIT_PATH` selects the toolkit. `CUDA_HOME` is a secondary option. The `tileiras` tool must be compatible with the selected toolkit; independently mixing versions is not a reproducible setup.

The CUDA driver is a separate host requirement. Installing toolkit files alone does not make GPU execution available.

## Official checkout checks

Within a configured CUDA Rust checkout, NVIDIA documents:

```sh
cargo run -p cutile-examples --example hello_world
cargo run -p cutile-examples --example saxpy
```

These commands were not run here. Record the checkout commit, toolkit, driver, and target GPU when establishing a working environment.

Sources: [cuTile setup](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/README.md#setup), [release notes](https://github.com/NVIDIA/cuda-rust/releases). Snapshot: live `main`, not release-pinned.
