# Awaiting Futures

```rust
use std::future::Future;

fn main() {
    println!("Hello, world!");
    let x = foo2();
}

async fn foo1() {
    println!("foo");
    0
}
// is the same as
fn foo2() -> impl Future<Output = ()>{
    async {
        println!("foo1");
        foo1.await();
        println!("foo2");
        0
    }
}
```
It will start running, it will print "foo1", then it will call foo1() and wait for it, then it will print "foo" and then "foo2".
If there was no await for "foo1", it would print "foo2" before "foo".

```rust
use std::future::Future;

fn main() {
    println!("Hello, world!");
    let x = foo2();
}

async fn foo1() {
    println!("foo");
    0
}
// is the same as
fn foo2() -> impl Future<Output = ()>{
    async {
        // First time:
        read_to_string("file1").await(); // wait here
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
While it is not ready it lets other things run (in terms of resources).
