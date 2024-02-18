# Cancellation
If you don't want a future to continue doing what it is doing, you can just drop it.
```rust
// cancellation chanel as an argument
fn foo(cancel: tokio::sync::mpsc::Receiver<()>) -> impl Future<Output = usize>{
    async {
        // First time:
        read_to_string("file1").await(); // no way to cancel as is
        println!("foo1");
        select! {
            done <- read_to_string("file2").await => {
                // continue; fall-through to print line below
            }
            cancel <- cancel.await => {
                return 0;
            }
        };

        // will not be executed if cancelled
        println!("foo1");
    }
}
```
You just describe circumstances under which you cancel the operation. "foo1" will not be printed in case of cancellation.
