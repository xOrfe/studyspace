---
title: Sources and Verification
tags: [rust, reference]
---

# Sources and Verification

## Language sources

Language notes explain stable Rust using edition 2024 syntax. Each page links to the relevant primary material:

- [The Rust Programming Language](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Standard library documentation](https://doc.rust-lang.org/std/)
- [Rust Reference](https://doc.rust-lang.org/reference/)
- [Cargo Book](https://doc.rust-lang.org/cargo/)
- [Async Book](https://rust-lang.github.io/async-book/)
- [Rustonomicon: FFI](https://doc.rust-lang.org/nomicon/ffi.html)

Examples are short explanations, not production application templates. A **complete example** contains an entry point or names its intended library file. A **snippet** requires surrounding context. **Pseudocode** explains an algorithm without claiming to be valid Rust or a particular crate API.

## GPU source snapshot

Verified on **2026-10-30** against:

| Source | How it is used |
| --- | --- |
| [CUDA Rust overview](https://nvidia.github.io/cuda-rust/) | Model comparison and shared runtime |
| [cuda-oxide book](https://nvidia.github.io/cuda-rust/cuda-oxide/) | Kernel, launch, safety, setup, and advanced concepts |
| [cuTile main README](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/README.md) | Setup and vector-add API pattern |
| [Official SAXPY example](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/cutile-examples/examples/saxpy.rs) | Broadcasting and tile loads/stores |
| [Official GEMM example](https://github.com/NVIDIA/cuda-rust/blob/main/cutile-rs/cutile-examples/examples/gemm.rs) | Tiled matrix computation and divisibility assumptions |
| [Release listings](https://github.com/NVIDIA/cuda-rust/releases) | Distinguishing live documentation from released versions |
| [CUDA Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/) | Memory and measurement background |

The source revision for repository examples is the mutable **`main` branch**, not an immutable commit. The live cuda-oxide book and repository are not assumed to describe the same release. Before building, select a checkout and follow its own manifest and toolchain files.

The cuTile hosted `latest` book could not be retrieved during authoring. Links to it are retained for reference; accessible official README and example files supplied the checked API details. No unverifiable reduction or boundary API has been invented: those algorithms are explicitly pseudocode.

## Local verification

- Checked notebook wiki links and local image references, Markdown fence pairing, frontmatter presence, and SVG XML validity.
- Rendered all five SVG diagrams at native width and at 390-pixel width; inspected the renders and a dark-background contact sheet. Diagrams use their own light background for consistent contrast.
- Reviewed the generated CPU/GPU illustration and its analogy disclaimer.
- **Rust examples were not compiled:** no Rust compiler was available. GPU code was not compiled or executed on hardware.
- **Quartz build was blocked:** dependencies are absent (`yargs` is the first missing package). The direct Node invocation was attempted. The npm script also uses a launch form that fails in this Windows shell; no existing scripts were changed.
- **Full-page desktop/mobile, light/dark, and Mermaid runtime rendering remain unverified** because the Quartz build could not run. Structural checks are not a substitute for those checks.

No toolchains or dependencies were installed, and the site was not published.

## Visual provenance

The SVG diagrams are original editable notebook assets. The CPU/GPU illustration was created with the built-in image generation tool and saved as `Rust/assets/cpu-gpu.png`.

<details>
<summary>Illustration generation prompt</summary>

Use case: scientific-educational. Create a polished editorial cut-paper/isometric illustration for an English Rust GPU notebook. Wide horizontal composition on warm off-white background. Two clearly separated workbenches: left a single versatile worker working sequentially on a short row of square mosaic pieces; right many simple identical workers simultaneously assembling separate small sections of a large square mosaic. Same teal, blue, and amber mosaic pieces on both sides. Communicate sequential versus massively parallel work on independent pieces; conceptual analogy, not actual chip architecture. Clean shapes, generous spacing, readable at small size. Only text: 'CPU' above left and 'GPU' above right. No code, no formulas, no logos, no extra labels, no watermark.

</details>

[[Rust/index|Rust notebook]]
