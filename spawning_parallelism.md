# Spawning and Parallelism
It is very inefficient to run futures concurrently in only one thread.
```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");

    let mut accept = tokio::net::TcpListener::bind("0.0.0.0:8080");

    let mut connections = futures::future::FuturesUnordered::new();
    loop {
        select! {
            // we need to process a new connection
            stream <- (&mut accept).await => {
                connections.push(handle_connection(stream));
            }
            // we need something to await connections
            _ <- (&mut connections).await => {
                // do something
            }
        }
    }
}

async fn handle_connection(_: TcpStream) {
    // do something with connection
}
```

In order to solve a single thread concurrent execution problems we may introduce parallelism.

```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");

    let mut accept = tokio::net::TcpListener::bind("0.0.0.0:8080");

    while let Ok(stream) = accept.wait {
        // this expression moves a future to an executor
        // it is not a thread spawn, runtime controls it
        // and spawn an additional thread since it has more
        // than one future
        tokio::spawn(handle_connection(stream));
    }
}

async fn handle_connection(_: TcpStream) {
    // do something with connection
}
```
In this case one thread handles all connections, another one handles a future for a particular connection. Spawn requires static future.

## Sharing Across Spawns

```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");

    let mut accept = tokio::net::TcpListener::bind("0.0.0.0:8080");

    while let Ok(stream) = accept.wait {
        tokio::spawn(handle_connection(stream));
    }
}

async fn handle_connection(_: TcpStream) {
    let x = Arc::new(Mutex::new(vec![]));
    let x1 = Arc::clone(&x);

    tokio::spawn(async move {
        x1.lock()
    });

    let x2 = Arc::clone(&x);
    tokio::spawn(async {
        x2.lock()
    });
}
```
Also we can use channels to communidcate to a future. There is no way to return an error from future to a part of the app that calls it and it is better to log an error or print it.