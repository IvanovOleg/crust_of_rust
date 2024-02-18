# Awaiting multiple futures
Imaging you have a bunch of futures. One f.e. waits for a user input in the terminal. Second one waits for the new connection.
You don't control those things. So we need to wait for both of them and we don't care who resolves first. Instead of spawning
threads for each operation, we can make it simplier by using **async/await**.

```rust
use std::future::Future;

fn main() {
    println!("Hello, world!");
    // it's ok to use threads, but it may get worse with spawning a thread for every operation we do
    let read_from_terminal = std::thread::spawn(move || {
        let mut x = std::io::Stdin::lock(std::io:stdin());
        for line in x.lines() {
            // do something on user input
        }
    });

    let read_from_network = std::thread::spawn(move || {
        let mut x = std::net::TcpListener::bind("0.0.0.0:8080").unwrap();

        while let Ok(Stream) = x.accept() {
            // do something on stream
            // now we have a single thread to manage all connections which is not great
            handle_connection(stream);
            // if one of clients is really slow, fast clients will have to wait until he receives a message anyway
            stream.write();
            // another option is to spawn a connection for each stream, but it will introduce complexity
            let handle = std::thread::spawn(move || {
                handle_connection(stream);
            });
        }
    });

    // Let's simplify, imaging we have functions somewhere (not as above), let's say these are futures (async functions)

    let network = read_from_network();
    let terminal = read_from_terminal();

    select! {
        stream <- network.await => {
            // do something on stream
        }
        line <- terminal.await => {
            // do something with line
        }
        foo <- foo2().await => {}
    };

    // or we can do it in the loop
    let mut foo = foo2();
    // if something happen in the !select, it returns and then
    // loop executes select again
    // we use mutable references to a future since  it will get
    // dropped after the first iteration of the loop
    loop {
        select! {
            stream <- (&mut network).await => {
                // do something on stream
            }
            line <- (&mut terminal).await => {
                // do something with line
            }
            foo <- (&mut foo).await => {}
        };
    }
}

async fn read_to_string(_: &str) {}
fn expensive_function(_: ()) {}
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
        println!("foo1");
        read_to_string("file4").await();
        println!("foo2");
        0
    }
}
```

**!select** macro that exists in a bunch of different libraries including **tokio**. It waits on a multiple futures and tells you whichever one finished first. 
In our example it tries to make progress on **network** and if it does, it will return a stream and call the code defined in the **network.await {}** block. 
If it doesn't make progress on **network**, it will try to make progress on **terminal**. If it makes progress on **terminal**, it will run the code in it's block.
If neither of them make progress, then it yields. And at some point in the future it will retry. In practice it is a lot smarter than this. So it yields until this
thing happens, not just yield. F.e. in case of network socket it will yield until somethin happens on that socket. It uses OS primitives to detect a potential reason to retry.
It's is not like a dumb loop behind the scene. It is smart enough not to retry if nothing changed on that socket. Select doesn't have a memory of it's past attempts,
that's why it selects all branches every time.
