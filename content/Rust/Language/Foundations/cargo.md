---
title: Cargo and Project Structure
tags: [rust, foundations]
---

# Cargo and Project Structure

`rustc` compiles Rust. **Cargo** manages packages, dependencies, builds, tests, and documentation. A package has a `Cargo.toml` manifest and can contain a library crate and executable crates. A **crate** is a compilation unit; a **module** organizes code inside one.

```text
hello/
├── Cargo.toml       package metadata and dependencies
├── Cargo.lock      resolved dependency versions
├── src/
│   ├── main.rs      executable crate root
│   └── lib.rs       optional library crate root
└── tests/          integration tests
```

Commands for an environment with Rust installed:

```sh
cargo new hello
cd hello
cargo check
cargo run
cargo build --release
cargo test
cargo doc --open
```

`check` checks the program without producing the final executable. `run` builds and executes the default binary. Release builds enable optimizations; debug builds favor quick compilation and debugging.

Minimal manifest:

```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2024"

[dependencies]
```

**Complete `src/main.rs`:**

```rust
fn main() {
    println!("Hello, Rust!");
}
```

`println!` is a macro; the `!` distinguishes it from a normal function call. The program prints `Hello, Rust!`.

An **edition** selects language compatibility rules; it is not the compiler version. A **workspace** groups packages under a shared manifest, lockfile, and build directory. Cargo features enable optional crate capabilities; they are not runtime switches.

Related: [[Rust/Language/Abstraction/modules|Modules and crates]] · [[Rust/Language/Advanced/testing|Testing and documentation]].

Sources: [Cargo Book](https://doc.rust-lang.org/cargo/), [packages and crates](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html).
