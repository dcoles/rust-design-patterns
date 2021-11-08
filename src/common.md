# Common Patterns

## New

**Idea:** Use an associated function as a type constructor.

```rust
struct MyType {
    value: Vec<u32>
}

impl MyType {
    /// Create a new [`MyType`].
    fn new() -> Self {
        MyType { value: Vec::new() }
    }
}

// A new instance of `MyType`
let my_class = MyType::new();
```

## Newtype

**Idea:** Wrap an existing type in a struct to form a new type.

This allows you to use Rust's type checker to ensure that the correct type is used.

```rust
struct AccountId(u64);

let account_id = AccountId(0x1234567812345678);
```

## Block expressions

**Idea:** Scope temporary variables in a block expression and return the result

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0u32));

let t = {
    // Temporary variables
    let counter = Arc::clone(&counter);

    // Block return value
    thread::spawn(move || {
        *counter.lock().unwrap() += 1;
    })
};

t.join().unwrap();
```

## Inner

**Idea:** Store a type's internal state in a private struct.

```rust
/// Public facing type.
pub struct Public(Inner);

/// Public API.
impl Public {
    pub fn new() -> Self {
        Public(Inner { state: 0 })
    }
}

/// Private inner state.
struct Inner {
    state: u32
}

/// Private API.
impl Inner {
    fn increment(&mut self) {
        self.state += 1;
    }
}
```
