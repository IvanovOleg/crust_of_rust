# Yielding
You can think of an async function as something that runs in chunks until it has to wait.
An then it yields (await). If there is a chain of awaits, then it yields to the top of
the chain, but next iteration of the loop starts with bottom.

```rust
fn foo2() -> impl Future<Output = ()>{
    async {
        // First time:
        // read_to_string("file1").await(); // wait here
        // abstract code
        let fut = read_to_string(file);
        let x = loop {
            if let Some(result) = fut.try_check_completed() {
                break result;
            } else {
                fut.try_make_progress();
                yeild; // yielding a thread (not only that, but you can think that)
            }
        };

        println!("foo1");
        // Second time:
        read_to_string("file2").await(); // wait here
        println!("foo1");
        read_to_string("file3");
        // without await does something like this (abstract code)
        while !fut.is_ready() {
            std::thread::yield_now();
            fut.try_complete();
        }
        //
        println!("foo1");
        read_to_string("file4").await();
        println!("foo2");
        0
    }
}
```