+++
title = 'Rust Async Programming'
date = 2023-12-09T23:40:15+08:00
lastmod = 2023-12-10T15:29:30+08:00
summary = "In-depth study and understanding of rust asynchronous programming."
tags = ["Rust", "Programming", "Async"]
+++

> Reference: [Rust Async Book](https://rust-lang.github.io/async-book/)

## Introduce to async

Async programming is a kind of concurrency programming. Different programming languages usually have different concurrency programming models for express concurrency.

Let's have a look at some other concurrency programming models first.

| Model        | Pros                                                                             | Cons                                                                                                                  |
| ------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| OS threads   | Doesn't depend on language model                                                 | Synchronizing between threads is difficult</br>Performance overhead is large, doesn't support a large number of tasks |
| Event-driven | In conjunction with callbacks, can be very performant                            | Result in a verbose, "non-linear" control flow. Makes it hard to follow data flow and error propagation               |
| Coroutines   | Like threads, don't depend on language model</br>Support a large number of tasks | The low-level details is abstracted away                                                                              |
| Actor model  | Can be efficiently implemented                                                   | Control flow and retry logic issue is unanswered                                                                      |

In summary, asynchronous programming can do: high performance, low-level support, and provides most of the ergonomic benefits of threads and coroutines.

## Async in Rust

Rust just provides async programming api, but not runtime, runtime is provided by community.
Meanwhile, the api is still growing, not mature. But they are stable enough to use and the official and community are working hard to mature them.

## Future

Future is the most important concept in rust asynchronous programming. It looks like this:

```rust
trait Future {
    type Output;
    fn poll(
        // Pin allows self reference, which is necessary to enable async/await.
        self: Pin<&mut Self>,
        // Used to wake up a specific task.
        cx: &mut Context<'_>,
    ) -> Poll<Self::Output>;
}
```

### Future Poll and Wakeups

Futures can be advanced by calling `poll` function. If the future comples, it returns `Poll::Ready`, otherwise it returns `Poll::Pending` and arranges for the `wake()` funtion to be called when the `Future` is ready to make more progess. When `wake()`, the executor knows exatly which futures are ready to be polled.

It's common that futures aren't able to return `Ready` the first time they are polled. In this case, the `Waker` type ensure this future will be polled again when it's ready to make more progress. `Waker` provides a `wake()` method, which is associated to an exatly `Future`. When `wake()` is called, the executor knows the future is ready to be polled again.

Here is an example of using `Waker`:

```rust
/// Spin up a new thread when the timer is created, sleep for the required time,
/// and then signal the timer future when the time windows has elapsed.
use std::{
    future::Future,
    pin::Pin,
    sync::{Arc, Mutex},
    task::{Context, Poll, Waker},
    thread,
    time::Duration,
};

pub struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>,
}

/// Shared state between the future and the waiting thread
struct SharedState {
    /// Whether or not the sleep time has elapsed
    ready: bool,

    /// The waker for the task that `TimerFuture` is running on.
    /// The thread can use this after setting `completed = true` to tell
    /// `TimerFuture`'s task to wake up, see that `completed = true`, and
    /// move forward.
    waker: Option<Waker>,
}

impl Future for TimerFuture{
    type Output = ();

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        // Look at the shared state to see if the timer has already completed.
        let mut shared_state = self.shared_state.lock().unwrap();

        if shared_state.ready {
            Poll::Ready(())
        } else {
            // Set waker so that the thread can wake up the current task
            // when the timer has completed, ensuring that the future is polled
            // again and sees that `completed = true`.
            //
            // It's tempting to do this once rather than repeatedly cloning
            // the waker each time. However, the `TimerFuture` can move between
            // tasks on the executor, which could cause a stale waker pointing
            // to the wrong task, preventing `TimerFuture` from waking up
            // correctly.
            //
            // N.B. it's possible to check for this using the `Waker::will_wake`
            // function, but we omit that here to keep things simple.
            shared_state.waker = Some(cx.waker().clone());
            Poll::Pending
        }

    }
}

impl TimerFuture{
    pub fn new(duration: Duration) -> Self {
        let shared_state = Arc::new(Mutex::new(SharedState{
            completed: false,
            waker: None,
        }));

        let thread_shared_state = shared_state.clone();
        thread::spawn(move || {
            thread::sleep(duration);
            let mut shared_state = thread_shared_state.lock().unwrap();
            // Signal that the timer has completed and wake up the last
            // task on which the future was polled, if one exists.
            shared_state.ready = true;
            if let Some(waker) = shared_state.waker.take() {
                waker.wake()
            }
        });

        TimerFuture { shared_state }
    }
}
```

## async/await Primer

`async` transform a block of code into a state machine that implements the `Future` trait.

When we want to synchronously run a future, we can use `block_on` function, but when the `future` is blocked synchronously, other functions can't attach to the thread until this future is finished.

We can introduce a `async runtime` to main function, block on it, using `await` to tell runtime when the `Future` is blocked, other `Future`s are able to attach to the thread.

## Runtime

We usually use tokio as async executor, but this time, we try to write our own runtime.

Future executors take a set of top-level `Future` and run them to completion by calling `poll` whenever the `Future` can make progress.

Here is an example of runtime:

```rust
use futures::{
    future::{BoxFuture, FutureExt},
    task::{waker_ref, ArcWake},
};
use std::{
    future::Future,
    sync::mpsc::{sync_channel, Receiver, SyncSender},
    sync::{Arc, Mutex},
    task::Context,
    time::Duration,
};
// The timer we wrote in the previous section:
use timer_future::TimerFuture;

struct Executor{
    ready_queue: Receiver<Arc<Task>>,
}

#[derive(Clone)]
struct Spawner{}
    task_sender: SyncSender<Arc<Task>>,
}

/// A future that can reschedule itself to be polled by an `Executor`.
struct Task{
    /// BoxFuture<T> is alias for Pin<Box<dyn Future<Output = T> + Send + 'static>>
    future: Mutex<Option<BoxFuture<'static, ()>>>>,

    /// Handle to place the task itself back onto the task queue.
    task_sender: SyncSender<Arc<Task>>,
}

fn new_executor_and_spawner() -> (Executor, Spawner) {
    // Maximum number of tasks to allow queueing in the channel at once.
    // This is just to make `sync_channel` happy, and wouldn't be present in
    // a real executor.
    const MAX_QUEUED_TASKS: usize = 10_000;
    let (task_sender, ready_queue) = sync_channel(MAX_QUEUED_TASKS);
    (Executor { ready_queue }, Spawner { task_sender })
}

impl Spawner {
    fn spawn(&self, future: impl Future<Output = ()> + 'static + Send) {
        let future = future.boxed();
        let task = Arc::new(Task {
            future: Mutex::new(Some(future)),
            task_sender: self.task_sender.clone(),
        });
        self.task_sender.send(task).expect("too many tasks queued");
    }
}

impl ArcWake for Task {
    fn wake_by_ref(arc_self: &Arc<Self>) {}
        // Implement `wake` by sending this task back onto the task channel
        // so that it will be polled again by the executor.
        let cloned = arc_self.clone();
        arc_self
            .task_sender
            .send(cloned)
            .expect("too many tasks queued");
    }
}

impl Executor {
    fn run(&self) {
        while let Ok(task) = self.ready_queue.recv() {
            let mut future_slot = task.future.lock().unwrap();
            if let Some(mut future) = future_slot.take() {
                let waker = waker_ref(&task);
                let context = &mut Context::from_waker(&*waker);
                if future.as_mut().poll(context).is_pending() {
                    *future_slot = Some(future)
                }
            }
        }
    }
}
```

Congratulations! We now have a working executor that can work with async/await.

```rust
fn main() {
    let (executor, spawner) = new_executor_and_spawner();
    spawner.spawn(async {
        println!("howdy!");
        TimerFuture::new(Duration::new(2, 0)).await;
        println("done!")
    });
    drop(spawner);
    executor.run();
}
```

## Executors and System IO

```rust
pub struct SocketRead<'a> {
    socket: &'a Socket,
}

impl SimpleFuture for SocketRead<'_> {
    type Output = Vec<u8>;

    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        if self.socket.has_data_to_read() {
            // The socket has data -- read it into a buffer and return it.
            Poll::Ready(self.socket.read_buf())
        } else {
            // The socket does not yet have data.
            //
            // Arrange for `wake` to be called once data is available.
            // When data becomes available, `wake` will be called, and the
            // user of this `Future` will know to call `poll` again and
            // receive data.
            self.socket.set_readable_callback(wake);
            Poll::Pending
        }
    }
}
```

For the code above, it's not clear how the `Socket` type is implemented, and in particular it isn't obvious how the `set_readable_callback` method works. How can we arrange for `wake` to be called once the socket becomes readable?

In practice, this problem is solved through integration with an IO-aware system blocking primitive, such as `epoll` on Linux, `kqueue` on FreeBSD and Mac OS, `IOCP` on Windows, and `port`s on Fuchsia. All of which are exposed through the cross-platform Rust crate [mio](https://github.com/tokio-rs/mio). These primitives all allow a thread to block on multiple asynchronous IO events, returning once one of the events completes. They usually look like this:

```rust
struct IoBlocker{
    // ... details elided ...
}

struct Event {
    // An ID uniquely identifying the event that occurred and was listened for.
    id: usize,
    // A set of signals to wait for, or which occurred.
    signals: Signals,
}

impl IoBlocker{
    fn new() -> Self {
        // ... details elided ...
    }

    fn add_io_event_interest(
        &self,
        /// The object on which the event will occur
        io_object: &IoObject,
        /// A set of signals that may apper on the `io_object` for
        /// which an event should be triggered, paired with
        /// an Id to give to events that result from these interests.
        signals: Signals
    ) {
        // ... details elided ...
    }

    /// Block until one of the events occurs.
    fn block(&self) -> Event {
        // ... details elided ...
    }
}

let mut io_blocker = IoBlocker::new();
io_blocker.add_io_event_interest(
    &socket_1,
    Event { id: 1, signals: READABLE },
);
io_blocker.add_io_event_interest(
    &socket_2,
    Event { id: 2, signals: READABLE | WRITABLE },
);
let event = io_blocker.block();
// prints e.g. "Socket 1 is now READABLE" if socket one became readable.
println!("Socket {:?} is now {:?}", event.id, event.signals);
```

Futures executor can use these primitives to provide asynchronous IO objects such as sockets that can configure callbacks to be run when a particular IO event occurs. In the case of our `SocketRead` example above, the `Socket::set_readable_callback` function might look like the following pseudocode:

```rust
impl Socket {
    fn set_readable_callback(&self, waker: Waker) {
        // `local_executor` is a reference to the local executor.
        // this could be provided at creation of the socket, but in practice
        // many executor implementations pass it down through thread local
        // storage for convenience.
        let local_executor = self.local_executor;

        // Unique ID for this IO object.
        let id = self.id;

        // Store the local waker in the executor's map so that it can be called
        // once the IO event arrives.
        local_executor.event_map.insert(id, waker);
        local_executor.add_io_event_interest(
            &self.socket_file_descriptor,
            Event { id, signals: READABLE },
        );
    }
}
```

We can now have just one executor thread which can receive and dispatch any IO event to the appropreate `Waker`, which will wake up the corresponding task, allowing the executor to drive more tasks to completion before returning to check for more IO events.
