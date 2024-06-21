# Brief Introduction to the Go Scheduler

Go's concurrency model, based on goroutines and the scheduler, is a powerful feature that makes writing concurrent programs easier and more efficient.

Goroutines and channels implement the Communicating Sequential Processes (CSP) model for concurrency, by using channels for data exchange between goroutines. They implement the userspace threads in Go, i.e., they are lightweight threads that are managed by the user-level runtime, in this case, the Go runtime, rather than the kernel. Features include:
- Lightweight because of small initial stack size (around 2KB), but as needed, a goroutine can dynamically adjust its stack size by releasing wasted memory or allocating more needed memory.
- Being userspace threads, they are faster to create and destroy.
- They are multiplexed into fewer kernel threads by the scheduler.
- No direct interaction with the kernel for context switching.

## What does the CSP Model Mean?

### Concurrency
Concurrency is a programming concept where multiple computations are executed during overlapping time periods concurrently instead of sequentially. It's not necessarily about doing many things at once, but about managing lots of things at once. Concurrency allows a program to make progress on several tasks seemingly at the same time, even if it runs on a single-core CPU by time-slicing. This means that the CPU switches between tasks so quickly that it gives the illusion that they are happening simultaneously.

Imagine you're cooking a meal: you might start chopping vegetables, then while they are sautéing, you boil water for pasta. You aren't doing all these tasks at the exact same time, but you're making progress on multiple tasks during the cooking process.

In programming, threads are often used to achieve concurrency. However, threads can be heavyweight, consuming significant resources because each thread requires its own stack and context. This is where goroutines come in: they are much lighter and more efficient than traditional threads, allowing thousands or even millions of concurrent tasks to be multiplexed on threads and managed with ease.

### CSP Model
The Communicating Sequential Processes (CSP) model describes patterns of interaction in concurrent systems. It is all about designing programs where components (processes) interact by passing messages through channels, rather than sharing memory.

In this model:
- **Processes** are independent entities that perform computations. In Go, goroutines are the processes.
- **Channels** are the roads through which processes communicate. Channels in Go provide a way for goroutines to send and receive data, ensuring safe communication and synchronization without needing explicit locks or condition variables.

The key idea in CSP is that processes should operate independently and communicate explicitly, making it easier to reason about their interactions and avoid issues like race conditions and deadlocks. Although there also exist the Actor Model for concurrency, Go implements the CSP model.

Okay, back to the Go scheduler.

## Distributed Runqueues

Go uses an M:N scheduling model, where N goroutines are mapped to M (machine, i.e. kernel) threads. This allows Go to have many more goroutines than there are available threads, making it possible to handle many concurrent tasks efficiently. `GOMAXPROCS` limits the number of threads that will be assigned to the Go runtime.

`GOMAXPROCS` is the number of OS threads that can execute user-level Go code simultaneously. It does not count the number of threads blocked in system calls.

In Go, each thread has its own local runqueue for managing goroutines. A runqueue is a queue of goroutines that are ready to run. When a thread claims a runqueue, it inserts and removes goroutines from it. If a local runqueue is empty, the thread can steal work from other overloaded threads' runqueues, balancing the workload across threads.

Local runqueues are associated, i.e. managed by the "p" structs. These structures also hold other essential resources required by a thread to execute goroutines, such as a memory cache. When a thread is blocked, the entire "p" is handed off in the handoff process.

Additionally, there exists a global runqueue, which serves as a low-priority queue for scheduling long-running or low-priority tasks.

The size of a local runqueue can be calculated by `(global_runqueue_length / GOMAXPROCS + 1)`. As of Go 1.22, the local runqueue has a limit of 256 goroutines (count of `ps` items = `GOMAXPROCS`).

### How Runqueues Work

#### Goroutine Creation:
When a new goroutine is created, it is placed into a runqueue. Typically, it goes into the local runqueue of the thread that created it.

#### Goroutine Scheduling:
- The scheduler picks goroutines from the runqueue to run on the available OS threads.
- If the local runqueue of a thread has goroutines, it schedules one of them to run.
- If the local runqueue is empty, the scheduler checks the global runqueue or steals from another thread's runqueue.

#### Goroutine Blocking:
If a goroutine performs a blocking operation (e.g., I/O, channel receive with no available data), it is removed from the runqueue and marked as waiting. Once the blocking operation completes, the goroutine is moved back to a runqueue.

### Example Scenario
Consider a program with several goroutines performing different tasks. Here's how runqueues might be used:
- Goroutine A, B, and C are created and placed in the local runqueue of Thread 1.
- Thread 1 starts executing Goroutine A.
- Meanwhile, Thread 2 has an empty local runqueue, so it steals Goroutine B from Thread 1's runqueue.
- Thread 2 starts executing Goroutine B, while Thread 1 continues with Goroutine A.
- Goroutine A completes its task and is removed from Thread 1's runqueue.
- If Goroutine C is still in Thread 1's runqueue, it will be scheduled next.
- If Thread 1's runqueue becomes empty, it will either pull from the global runqueue or steal from another thread's runqueue.

### Thread Blocking
If a goroutine calls a blocking system call, we don't want the thread's runqueue to idle and starve the goroutines. So, we have a background mechanism called "handoff" which assigns the blocked thread's runqueue to another thread. Handoff unparks a parked thread or starts a thread if necessary to assign runqueue of blocked thread.

You might wonder, why doesn't blocked thread itself doesn't perform handoff of the runqueue by itself? Handoffs can get expensive, especially when new thread is started. Therefore, immediate handoffs are only performed for specific system calls. If the blocking period is expected to be very short, it can be more efficient to wait rather than initiate a handoff. If both the goroutine and the thread are blocked and a handoff hasn't occurred yet, sysmon comes to the rescue and initiates a handoff.

When the system call returns, the scheduler tries to reschedule the goroutine on its original thread. If that's not possible, it attempts to assign it to an idle thread. If no idle thread is available, the scheduler places the goroutine in the global queue and parks the thread that was involved in the system call.

### Thread Spinning
To optimize the process of parking and unparking threads, threads without work "spin" looking for work before parking. They check the global runqueue. If the global runqueue is empty, it checks on the netpoller if it has a goroutine in a runnable state and starts executing that (netpoller handles network I/O, e.g. read, write, connect, accept, close). It will also attempt to run GC tasks or work steal.

## Preemption

Go's scheduler initially used cooperative preemption, where goroutines voluntarily yield control at certain points. in their execution. But what if there is a program that is CPU-intensive, has no blocking work, and doesn't create goroutines? It will starve the local runqueue. We need a way to make this program yield control to the Go scheduler. For this, the Go scheduler implements non-cooperative preemption, i.e., a background thread called sysmon detects long-running goroutines and unschedules them when possible. It puts those preempted goroutines into the global runqueue (lower priority queue, so contention is not a problem).

Runqueues in go scheduler primarily use FIFO (First in, First out) queues. It is good for fairness, but bad for locality.

#### Fairness
Fairness in scheduling means that each goroutine gets a fair share of CPU time over a period. In a FIFO (First in, First out) queue, fairness implies that goroutines are executed in the order they arrive. This ensures that no goroutine is indefinitely delayed or starved of CPU time, assuming there are multiple goroutines waiting to be scheduled. Fair scheduling is beneficial for scenarios where multiple goroutines are competing for resources and need to make progress without one dominating the CPU for extended periods.

#### Locality
On other hand, Locality in scheduling refers to the tendency of keeping related tasks (here, goroutines) close to each other in the execution order. This can lead to better CPU cache utilization and reduced overhead related to task switching. In LIFO (Last in, First out) queue, locality implies that recently added goroutines are prioritized for execution. This can be advantageous in scenarios where tasks are dependent on each other or where the most recently added tasks are more likely to benefit from cache locality, improving overall execution efficiency. Hence, LIFO queues implement locality better, but they are bad in fairness as newer tasks are prioritized, potentially delaying older tasks indefinitely if new tasks keep arriving. 

In this blog, I tried my best to explain some concepts that is used in Go scheduler.

Thankyou for reading my blog.