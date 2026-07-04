# 🚦 Semaphores

---

## What is a Semaphore?

A **semaphore** is an integer variable that is accessed only through two **atomic** operations: `wait()` and `signal()`.

> **Analogy**: A parking lot with N spots. A counter tracks free spots. Cars (processes) check the counter before entering; they signal when leaving.

```
Semaphore S = N    (N = number of available resources/permits)

wait(S):    Decrement S. If S < 0, BLOCK the process.
signal(S):  Increment S. If any process is blocked, WAKE one up.
```

---

## ⚙️ wait() and signal() Operations

### Classic Definitions

```c
// wait() — also called P() or down()
wait(S) {
    S = S - 1;
    if (S < 0) {
        // Add this process to S's waiting queue
        block();   // sleep until woken
    }
}

// signal() — also called V() or up()
signal(S) {
    S = S + 1;
    if (S <= 0) {
        // Remove one process from S's waiting queue
        wakeup(waiting_process);
    }
}
```

> **Both operations must be atomic** — no interruption between the check and the modify.

### The Sign of S Tells You:

| S value | Meaning |
|---------|---------|
| S > 0 | That many resources are available |
| S = 0 | No resources free, no one waiting |
| S < 0 | \|S\| processes are currently BLOCKED waiting |

---

## 1. 🔴 Binary Semaphore

A semaphore initialized to **1**.  
Behaves exactly like a mutex — only **0** or **1**.

```
S = 1 (initially)

Process A calls wait(S):  S becomes 0 → A enters CS
Process B calls wait(S):  S becomes -1 → B BLOCKS
Process A calls signal(S): S becomes 0 → B is WOKEN → B enters CS
```

### Usage Pattern

```c
semaphore mutex = 1;   // binary semaphore

// Process:
wait(mutex);
//  ─── CRITICAL SECTION ───
signal(mutex);
```

**Difference from Mutex**: A semaphore has **no concept of ownership**. Any process can call `signal()`, even if it didn't call `wait()`. A mutex must be unlocked by the same thread that locked it.

---

## 2. 🟢 Counting Semaphore

A semaphore initialized to **N** (any positive integer).  
Controls access to **N identical resources** (e.g., N database connections, N printer slots).

```
S = 3   (3 resources available)

P1 calls wait(S): S = 2 → P1 enters
P2 calls wait(S): S = 1 → P2 enters
P3 calls wait(S): S = 0 → P3 enters
P4 calls wait(S): S = -1 → P4 BLOCKS (no resources!)
P1 calls signal(S): S = 0 → P4 WOKEN → P4 enters
```

### Usage Pattern

```c
semaphore resource_count = 3;   // 3 resources

// Any process wanting a resource:
wait(resource_count);
//  use resource
signal(resource_count);
```

---

## Binary vs Counting Semaphore

| Feature | Binary Semaphore | Counting Semaphore |
|---------|-----------------|-------------------|
| **Initial value** | 1 | N (any positive int) |
| **Range** | 0 or 1 | 0 to N |
| **Purpose** | Mutual exclusion | Resource counting / limiting |
| **Similar to** | Mutex (but no ownership) | Resource pool |
| **Example** | Protecting one shared variable | Connection pool (N=10) |

---

## 🔄 Producer-Consumer Problem (Classic Example)

**Problem**: Producer adds items to buffer, Consumer removes them.  
Buffer has fixed size N. Must not overflow or underflow.

```c
semaphore empty = N;    // empty slots (initially all N)
semaphore full  = 0;    // filled slots (initially none)
semaphore mutex = 1;    // mutual exclusion on buffer

// PRODUCER:
while (true) {
    produce_item();
    wait(empty);          // wait for an empty slot
    wait(mutex);          // lock buffer
    add_to_buffer();
    signal(mutex);        // unlock buffer
    signal(full);         // signal one more item available
}

// CONSUMER:
while (true) {
    wait(full);           // wait for an item
    wait(mutex);          // lock buffer
    remove_from_buffer();
    signal(mutex);        // unlock buffer
    signal(empty);        // signal one more empty slot
    consume_item();
}
```

**Trace** (N=2, 1 producer, 1 consumer):

```
empty=2, full=0, mutex=1 (initial)

Producer: wait(empty)→empty=1, wait(mutex)→mutex=0, add item, signal(mutex)→mutex=1, signal(full)→full=1
Consumer: wait(full)→full=0, wait(mutex)→mutex=0, remove item, signal(mutex)→mutex=1, signal(empty)→empty=2
```

---

## Semaphore for Signaling (Not Just Mutual Exclusion!)

Semaphores can also be used for **ordering / signaling** between processes:

```c
semaphore sync = 0;   // initially 0

// Process A:
// Do some work...
signal(sync);         // tell B that work is done

// Process B:
wait(sync);           // wait for A to finish
// Now B can proceed  // guaranteed to run AFTER A's signal
```

This ensures Process B always executes **after** Process A's signal — useful for ordering.

---

## ⚠️ Semaphore Pitfalls

| Mistake | Consequence |
|---------|-------------|
| `wait(S)` but forget `signal(S)` | Deadlock — no one can enter CS |
| `signal(S)` before `wait(S)` | Two processes in CS simultaneously |
| Wrong order of `wait(mutex)` and `wait(empty/full)` | Deadlock |
| Using semaphore with initial value 0 for mutex | Deadlock immediately |

---

## 🎯 Interview Questions & Answers

**Q: What is a semaphore?**
> A semaphore is an integer variable accessed through two atomic operations: `wait()` (decrement, block if < 0) and `signal()` (increment, wake a blocked process if any). It's used for mutual exclusion and process synchronization.

**Q: What is the difference between a binary semaphore and a counting semaphore?**
> A binary semaphore has initial value 1 and acts like a mutex — only one process in the CS at a time. A counting semaphore has initial value N and controls access to N identical resources, allowing up to N processes to proceed concurrently.

**Q: What is the difference between a semaphore and a mutex?**
> A mutex has ownership — only the thread that locked it can unlock it. A semaphore has no ownership — any process can call signal(). Semaphores can also be used for signaling between processes (initial value 0), not just mutual exclusion.

**Q: Explain the Producer-Consumer problem and how semaphores solve it.**
> Producer adds to a bounded buffer; Consumer removes from it. Without sync, producer can overflow the buffer or consumer can read empty buffer. Semaphores: `empty` (N, counts empty slots), `full` (0, counts filled slots), `mutex` (1, for buffer access). Producer waits for empty slot, consumer waits for full slot — they signal each other.

**Q: What are wait() and signal() operations?**
> `wait(S)`: Decrements S. If S < 0, the calling process blocks (sleeps). `signal(S)`: Increments S. If S ≤ 0, one blocked process is woken up. Both operations must execute atomically.

---

*← [Mutex and Spinlocks](./05-Mutex-and-Spinlocks.md) | Next → [Deadlocks](./07-Deadlocks.md)*
