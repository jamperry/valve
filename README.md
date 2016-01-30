# valve
Application framework in Rust to build asynchronous services rapidly.

## Architecture

valve is built on top of [MIO]() to facilitate an OS agnostic framework.The main event loop handles incoming connections and it schedules connections to workers to process requests and send responses.

The server will set the underlying process' file limit to the maximum number possible (only *nix for the time being) and then pre-allocates a connection array to the length of this upper bound for performance. The connection array represents file descriptors and passes them to the workers. By pre-allocating it minimizes memory fragmentation and the overhead of `malloc`.

Threads are locked to the CPU cores using thread affinity to improve cache locality and minimize OS thread switching. Thread switching is quite cheap - about the cost of a function call - but the problem is that OS thread switching is a butterfly effect; it will typically re-schedule the other n-1 threads so it is significant if there are lot of cores.

It uses fibers to model the asynchronous IO. They are cheap to create since it is just a `malloc` (can be proallocated) and even cheaper to schedule because it is just a stack/instruction register swap - which is cheaper than a thread switch.

## Typed RPC

The main focus for the alpha is to support a strongly typed RPC system influenced by [Yaron Minsky](https://blogs.janestreet.com/typing-rpcs/) of Jane Street. It takes advantage of Rust's efficient pattern matching to send recursive ADTs safely. It facilitates to send recursive data structures (like a AST, graph, etc) efficiently and safely.

Here's an example of typed messages for a basic File Service.

```rust
// request enumeration to represent an request message for a file service
enum Req {
  ListAll,
  Size(Path),
  Get(Path),
  Exists(Path),
  Del(Path),
}

// reply enumeration to represent a reply message 
enum Rep {
  Ok,
  List(Vec<File>),
  Content(String),
  Size(usize),
  Exists(bool),
  Error(String),
}
```
