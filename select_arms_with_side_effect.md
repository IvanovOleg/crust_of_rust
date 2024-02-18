# Select Arms With Side Effect
```rust
#[tokio::main]
async fn main() {
    let network = read_from_network();
    let terminal = read_from_terminal();

    // we use tokio::fs instead of standard library since we need async operation
    // In standard library all IO operations are sync
    let mut f1 = tokio::fs::File::open("foo");
    let mut f2 = tokio::fs::File::create("bar");
    let copy = tokio::io::copy(&mut f1, &mut f2);

    select! {
        stream <- network.await => {
            // do something on stream
        }
        line <- terminal.await => {
            // do something with line
        }
        _ <- copy.await => {}
    };

    // some bytes have been copied from foo to bar, but not all (current state)
    // since select! is not inside the loop
    // copy.await; we may easily forget to await and got a partially completed future
    // That partially copied file is called side-effect.
}
```