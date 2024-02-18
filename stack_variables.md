# Stack Variables
State of the stack is saved in a specific storage similar to enum and in our example "x" doesn't disappear when
a future yields (returns something).
```rust
use std::future::Future;

#[tokio::main]
async fn main() {
    let mut x = foo(); // the value of "x" is of type StateMachine
    x.await; // some sort of StateMachine::await(&mut x)
}

async fn foo() {
    let mut x = [0; 1024]; // byte array
    let n: usize = tokio::fs::read_into("file.dat", &mut x[..]).await;
    println!("{:?}", x[..n]);
}
```