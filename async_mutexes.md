# Asynchronous Mutexes
Using a standard libary mutex in a single thread executed can create a dead lock.
```rust
use std::future::Future;
use std::sync::{Arc, Mutex};


pub mod tokio {
    pub async fn spawn(_: impl Future) {}
    pub mod sync {
        pub struct Mutex;
    }
}

use tokio::sync::Mutex as TMutex;

async fn main() {
    let x = Arc::new(Mutex::new(0));
    let x1 = Arc::clone(&x);
    // we have two aync threads, they both accessing the same arc
    // but through a mutex
    tokio::spawn(async move {
        loop {
            // this lock never releases
            let x = x1.lock();
            tokio::fs::read_to_string("file").await;
            *x += 1;
            *x1.lock() += 1;
        }
    });
    
    tokio::spawn(async move {
        loop {
            // this lock never completes
            *x2.lock() -= 1;
        }
    });
}

```

```rust
use std::future::Future;
use std::sync::{Arc, Mutex};


pub mod tokio {
    pub async fn spawn(_: impl Future) {}
    pub mod sync {
        pub struct Mutex;
    }
}

use tokio::sync::Mutex as TMutex;

async fn main() {
    let x = Arc::new(TMutex::new(0));
    let x1 = Arc::clone(&x);
    // we have two aync threads, they both accessing the same arc
    // but through an async mutex
    tokio::spawn(async move {
        loop {
            // this will process
            let x = x1.lock();
            tokio::fs::read_to_string("file").await;
            *x += 1;
            *x1.lock() += 1;
        }
    });
    
    tokio::spawn(async move {
        loop {
            // this will yield until lock is release since it is aware
            *x2.lock() -= 1;
        }
    });
}
```
That's why we need async enabled mutexes. The downside of these mutexes is they significantly slower than original. Use standard library
mutexes (in non-critical sections or when critical section is short and there is no await) unless it is absolutely required to use async enabled.
