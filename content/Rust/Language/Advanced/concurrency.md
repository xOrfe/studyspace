---
title: Threads, Channels, Mutex, Send, and Sync
tags: [rust, advanced, concurrency]
---

# Threads, Channels, Mutex, Send, and Sync

**Concurrency** organizes overlapping work; **parallelism** executes work simultaneously. Rust's ownership system helps prevent data races, but it does not prevent deadlocks or incorrect algorithms.

**Complete example:**

```rust
use std::{sync::{Arc, Mutex}, thread};

fn main() {
    let count = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    for _ in 0..2 {
        let shared = Arc::clone(&count);
        handles.push(thread::spawn(move || {
            *shared.lock().unwrap() += 1;
        }));
    }
    for handle in handles { handle.join().unwrap(); }
    assert_eq!(*count.lock().unwrap(), 2);
}
```

`Arc` shares ownership. `Mutex` grants one lock holder access to the integer at a time. Its guard unlocks when dropped. `join` waits for each thread and reports if it panicked. The unwraps here keep a small demonstration focused; production error handling depends on the application.

## Send and Sync

| Trait | Meaning |
| --- | --- |
| `Send` | Ownership of a value can be transferred between threads |
| `Sync` | Shared references to a value can be transferred between threads: `&T` is Send |

These are unsafe auto traits, usually inferred from a type's fields. `Rc` is neither Send nor Sync. `Arc` only supports cross-thread sharing when its contents meet the required bounds. Manually implementing these traits requires proving their contracts.

## Channels and locks

`std::sync::mpsc::channel()` sends owned messages from producers to a receiver. Channels transfer information; locks coordinate access to shared state. Sending an owned `String` moves it into the channel.

Keep lock scopes short and acquire multiple locks in a consistent order. A mutex may become poisoned after a panic while held; poisoning is an advisory error condition, not memory-safety enforcement.

Related: [[Rust/Language/Advanced/async|Async]] · [[Rust/GPU/foundations/streams|GPU streams]].

Sources: [Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html), [Send and Sync](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html).
