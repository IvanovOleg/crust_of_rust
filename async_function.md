# Async Function
These two async function definitions are similar and basically one is converded to another one.
```rust
use std::future::Future;

fn main() {
    println!("Hello, world!");
}

async fn foo1() {} // async keyword here is just a transformation keyword for the compiler
// is the same as
fn foo2() -> impl Future<Output = ()>{
    async {}
}
```
We return some type that implements the future trait.
```rust
#[allow(dead_code, unused_variables)]
use std::future::Future;

fn main() {
    println!("Hello, world!");
    let x: usize = foo3(); // will not compile since it is not a usize
    let y: usize = foo3().await; // since we were waiting for a result and result is usize then it will compile.
}

async fn foo3() -> usize {
    0
}

fn foo4() -> impl Future<Output = (usize)> {
    async { 0 }
}
```
Future represents a value which is not ready yet. In our example that value will eventually be a **usize**.
*await* means we don't run the following list of instructions until the expression is resolved to it's output type.

```rust
#[allow(dead_code, unused_variables)]
use std::future::Future;

fn main() {
    let x = foo();
}

async fn foo() -> usize {
    println!("Foo"); // Foo will not be printed since future does nothing until it is awaited
    0
}
```
Future just describes a series of steps that will be executed at some point. The first time the **foo** function gets awaited it will print "Foo" and return a 0 as a result.