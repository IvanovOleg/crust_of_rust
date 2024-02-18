# Async Function In Trait

```rust
use std::future::Future;

fn main() {

}

struct Request;
struct Response;

trait Service {
    // currently this dis not supported
    async fn call(&mut self, _: Request) -> Response;
}

struct X;
impl Service for X {
    async fn call(&mut self, _: Request) -> Response {
        Response
    }
}

struct Y;
impl Service for Y {
    async fn call(&mut self, _: Request) -> Response {
        let z = [0; 1024];
        tokio::time::sleep(100).await;
        drop(z);
        Response
    }
}

// fut has no known size, compiler doesn't know it, we need to use async_trait crate
fn foo(x: &mut dyn Service) {
    let fut = x.call(Request);
}
```
**#[async_trait]** rewrites all of your async events to:
```rust
use std::future::Future;

fn main() {

}

struct Request;
struct Response;

#[async_trait]
trait Service {
    fn call(&mut self, _: Request) -> Pin<Box<dyn Future<Output = Response>>>;
}

struct X;
impl Service for X {
    async fn call(&mut self, _: Request) -> Pin<Box<dyn Future<Output = Response>>> {
        Box::pin(async move { Response })
    }
}

struct Y;
#[async_trait]
impl Service for Y {
    async fn call(&mut self, _: Request) -> Response {
        let z = [0; 1024];
        tokio::time::sleep(100).await;
        drop(z);
        Response
    }
}

// fut has no known size, compiler doesn't know it, we need to use async_trait crate
fn foo(x: &mut dyn Service) {
    let fut = x.call(Request);
}
```
The problem is now you heap allocate all of your futures.

Another option:
```rust
use std::future::Future;

fn main() {

}

struct Request;
struct Response;

trait Service {
    type CallFuture: Future<Output = Response>;
    fn call(&mut self, _: Request) -> Self::CallFuture;
}

struct X;
impl Service for X {
    type CallFuture = Pin<Box<dyn Future<Output = Response>>>;
    fn call(&mut self, _: Request) -> Self::CallFuture {
        Box::pin(sync move { Response })
        //or
        // async { Response }
    }
}
```