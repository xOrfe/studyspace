---
title: cuda-oxide Environment and Setup
tags: [rust, gpu, cuda-oxide, setup]
---

# cuda-oxide Environment and Setup

This page describes the supported environment; it does not install anything.

The live installation guide checked on **2026-10-30** lists Linux (Ubuntu 24.04 tested), an Ampere-or-newer GPU, CUDA Toolkit 13.0+, LLVM/Clang 21+, and a pinned Rust 1.99.1 toolchain managed by `cargo oxide`. It explicitly says native Windows is unsupported. Recheck the guide before selecting versions.

The driver and toolkit are separate. A sufficiently new toolkit can still produce PTX that an older driver cannot JIT. Architecture-specific features can require newer hardware than the baseline.

## Reproducible environment

NVIDIA supplies a devcontainer containing the compiler dependencies. The host still needs a compatible NVIDIA driver and GPU-enabled container setup. Its Nix configuration is another supported route. Use the official setup instructions for the selected checkout rather than mixing installation steps across releases.

After configuring the official environment, the documented checks include:

```sh
cargo oxide doctor
cargo oxide run vecadd
```

`doctor` diagnoses the selected tools; `run` builds and executes an example. These commands are reference instructions and were not executed here.

Record the checkout commit, driver, toolkit, and GPU architecture with any working setup.

Source: [Official installation guide](https://nvidia.github.io/cuda-rust/cuda-oxide/getting-started/installation.html). Snapshot: live documentation, not a pinned release.
