# Executing Futures
We can't put everything in the future. Something should execute it. That's why rust doesn't allow you to make the main function async.
This happens because future does nothing by itself. So we need an executor. Executor keeps retrying argessively.
```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");
}

// that attribute converts function to:
fn main() {
    let runtime = tokio::runtime::Runtime::new(); // creates that big loop that polls the future
    runtime.block_on(async {
        println!("Hello, world!");
    });
}
```
Executors puts everything into sleep and tells OS to wake it up on certain events. It runs available futures until there is no more
futures to run.