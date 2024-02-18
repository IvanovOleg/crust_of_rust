# Awaiting Multiple Futures
We can use joins in order to ask to wait until all of these futures complete.
F.e let's say you want to read 10 files.

```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");

    // iterator of futures and we need to await all of them
    let files: Vec< > = (0..3).map(|i| tokio::fs::File::read_to_string(format!("file{}", i))).collect();
    // compare this
    let file1 = flies[0].await;
    let file2 = flies[1].await;
    let file3 = flies[2].await;
    // to this
    let (file1, file2, file3) = join!(files[0], files[1], files[2]);
}
```
If we have separate awaits, we first read file1 to completeon, then switch to file2, then to file3. It works, but it is sequential.

Join lets you run all reads concurrently and they all completed, it will return them at the same time.

If you have a lot of futures to join, you can try to use
**futures::future::try_join_all** method.