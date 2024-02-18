# Future Tracing
We can use the tracing_futures crate.
```rust
use std::future::Future;
use tracing_futures::Instrument;


fn main () {}

async fn foo() {
    tokio::spawn(async {
        tracing::error!("your error");
    }).instrument("in foo"); // takes a future and returns a future
}
```
