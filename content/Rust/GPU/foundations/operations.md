---
title: Three Common GPU Operations
tags: [rust, gpu, algorithms]
---

# Three Common GPU Operations

These operations illustrate independent work, cooperation, and data reuse. The arithmetic stays the same across CPU, SIMT, and tile implementations.

## Vector addition

```text
A:  1   2   3   4
B: 10  20  30  40
C: 11  22  33  44
```

Each `C[i] = A[i] + B[i]` is independent. SIMT can assign one element to each thread; tile programming can assign a group of elements to each program instance.

## Reduction

Combine a sequence into fewer values, often one sum:

```text
1    2    3    4
 \  /      \  /
  3         7
    \     /
       10
```

Threads or tiles compute partial sums, then another stage combines them. A floating-point tree can produce a different rounding result than a left-to-right sum because addition is not associative in floating-point arithmetic.

## Matrix multiplication

For `C = A × B`, `C[row,col]` is the dot product of a row of A and a column of B.

```text
A = [1 2]    B = [5 6]    C = [19 22]
    [3 4]        [7 8]        [43 50]

C[0,0] = 1*5 + 2*7 = 19
```

**Complete CPU example:**

```rust
fn main() {
    let a = [[1, 2], [3, 4]];
    let b = [[5, 6], [7, 8]];
    let mut c = [[0; 2]; 2];
    for row in 0..2 {
        for col in 0..2 {
            for k in 0..2 {
                c[row][col] += a[row][k] * b[k][col];
            }
        }
    }
    assert_eq!(c, [[19, 22], [43, 50]]);
    let added: Vec<_> = [1, 2, 3, 4].iter()
        .zip([10, 20, 30, 40]).map(|(x, y)| x + y).collect();
    assert_eq!(added, vec![11, 22, 33, 44]);
    assert_eq!([1, 2, 3, 4].iter().sum::<i32>(), 10);
}
```

Related: [[Rust/GPU/cuda-oxide/reduction-matmul|SIMT reductions and matrices]] · [[Rust/GPU/cutile-rs/reduction-matmul|Tile reductions and matrices]].

Source: [CUDA Best Practices: matrix multiplication](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html#memory-optimizations).
