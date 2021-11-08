# Multithreading Patterns

Multithreading in Rust introduces some unique challenges. Unlike many other lanaguages, the compiler is strict on enforcing safe access to shared values.

## Move

**Idea:** Move ownership of a value exclusively to a new thread.

```rust
use std::thread;

let values = vec![1, 2, 3, 4, 5];

let t = thread::spawn(move || {
    // Thread takes exclusive ownership of `values`
    for value in values {
        println!("{}", value);
    }
});

t.join();
```

## Threadsafe

**Idea:** Make a type threadsafe by protecting the internal state with a [`Mutex`](https://doc.rust-lang.org/std/sync/struct.Mutex.html).

This type implements [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html), so it can be easily shared between threads.

```rust
use std::sync::{Arc, Mutex};

#[derive(Clone)]
struct Counter(Arc<Mutex<u64>>);

impl Counter {
    fn new() -> Self {
        // Internal state is protected by a Mutex
        Counter(Arc::new(Mutex::new(0)))
    }

    fn value(&self) -> u64 {
        *self.0.lock().unwrap()
    }

    fn increment(&self) {
        *self.0.lock().unwrap() += 1;
    }
}
```

## Handle

**Idea:** Split a class into an exclusive owned type and a thread-safe handle.

```rust
use std::io;
use std::sync::mpsc;

/// Worker with exclusive access to a resource.
struct Worker {
    channel: mpsc::Receiver<Message>
}

impl Worker {
    /// Returns ([`Worker`], [`Handle`]) pair.
    fn new() -> (Self, Handle) {
        // Channel for sending messages to the worker.
        let (tx, rx) = mpsc::channel();

        (Worker { channel: rx }, Handle { channel: tx })
    }

    /// Function which takes exclusive access of [`Worker`].
    fn run(&mut self) {
        loop {
            while let Ok(message) = self.channel.recv() {
                match message {
                    Message::Foo(value) => println!("Foo: {}", value),
                }
            }
        }
    }
}

/// Handle for interfacing with the worker.
#[derive(Clone)]
struct Handle {
    channel: mpsc::Sender<Message>
}

impl Handle {
    fn foo(&self, value: i32) {
        self.channel.send(Message::Foo(value)).unwrap()
    }
}

/// Messages that can be sent to [`Worker`].
enum Message {
    Foo(i32)
}
```
