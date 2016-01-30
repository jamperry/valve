# valve
Application framework in Rust to build asynchronous services rapidly.

## Architecture

valve is built on top of [MIO]() to facilitate an OS agnostic framework.The main event loop handles incoming connections and schedules connections to workers to process requests and send responses.

The server will set file limit to maximum number possible for the underlying OS and pre-allocate a connection array to the length of this upper bound for performance. The connection array is a Vec<i32> to represent file descriptors and passes them to the workers. By pre-allocating it minimizes memory fragmentation and the overhead of `malloc`.

Threads are locked to the CPU cores using thread affinity to improve cache locality and minimize OS thread switching. Thread switching is quite cheap - about the cost of a function call - but the problem is that OS thread switching is a butterfly effect; it will typically re-schedule the other n-1 threads so it is significant if there are lot of cores.

It uses fibers to model the asynchronous IO. They are cheap to create since it is just a `malloc` (can be proallocated) and even cheaper to schedule because it is just a stack/instruction register swap - which is cheaper than a thread switch.

## Strongly typed RPC

The main focus for the alpha is to support a strongly typed RPC system to take advantage of Rust's efficient pattern matching to send recursive ADTs.

```rust
// request enumeration to represent an messages for a file service
enum Req {
  ListAll,
  Size(Path),
  Get(Path),
  Exists(Path),
  Del(Path),
}

enum Rep {
  List(Vec<File>),
  Content(String),
  Size(usize),
  Exists(bool),
  Error(String),
}
```
