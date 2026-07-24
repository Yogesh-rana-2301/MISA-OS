# Classic Synchronization Problem 1 — Producer-Consumer

---

## Problem Statement

- A **producer** generates data items and places them in a **shared bounded buffer**.
- A **consumer** takes items from the buffer and processes them.
- Buffer has fixed capacity **N**.

**Three constraints:**
1. Producer must not add to a **full** buffer.
2. Consumer must not remove from an **empty** buffer.
3. Only one process can access the buffer at a time (**mutual exclusion**).

```
Producer ──[item]──→ [ _ | _ | A | B | C ] ──[item]──→ Consumer
                       bounded buffer (capacity N=5)
                       currently: 3 items, 2 empty slots
```

---

## Solution Using Semaphores

Three semaphores:

| Semaphore | Initial Value | Meaning |
|-----------|--------------|---------|
| `empty` | N | Number of empty slots available |
| `full` | 0 | Number of filled slots available |
| `mutex` | 1 | Binary semaphore for buffer mutual exclusion |

```c
semaphore empty = N;   // N empty slots initially
semaphore full  = 0;   // 0 filled slots initially
semaphore mutex = 1;   // binary lock on buffer

int buffer[N];
int in = 0, out = 0;  // producer writes at 'in', consumer reads at 'out'

// PRODUCER:
void producer() {
    while (true) {
        int item = produce_item();

        wait(empty);      // wait if no empty slot (blocks if empty == 0)
        wait(mutex);      // lock buffer

        buffer[in] = item;
        in = (in + 1) % N;

        signal(mutex);    // unlock buffer
        signal(full);     // one more item available for consumer
    }
}

// CONSUMER:
void consumer() {
    while (true) {
        wait(full);       // wait if no item (blocks if full == 0)
        wait(mutex);      // lock buffer

        int item = buffer[out];
        out = (out + 1) % N;

        signal(mutex);    // unlock buffer
        signal(empty);    // one more empty slot available for producer

        consume_item(item);
    }
}
```

---

## Step-by-Step Trace (N=3)

Initial: `empty=3, full=0, mutex=1`, buffer = [_, _, _]

```
Producer runs:
  wait(empty) → empty=2
  wait(mutex) → mutex=0
  buffer[0] = A, in=1
  signal(mutex) → mutex=1
  signal(full) → full=1
  Buffer: [A, _, _]

Producer runs again:
  wait(empty) → empty=1
  ... buffer[1] = B, full=2
  Buffer: [A, B, _]

Consumer runs:
  wait(full) → full=1
  wait(mutex) → mutex=0
  item = A, out=1
  signal(mutex) → mutex=1
  signal(empty) → empty=2
  Buffer: [A, B, _]  (A consumed, slot freed)

Producer tries when buffer full (empty=0):
  wait(empty) → empty=-1 → BLOCKED (waits for consumer to signal)
```

---

## Critical Order: Why wait(mutex) AFTER wait(empty/full)?

**Wrong order (causes deadlock):**
```c
// WRONG — deadlock possible!
wait(mutex);    // lock buffer first
wait(empty);    // then wait for slot — but mutex is held!

// If producer holds mutex and waits for 'empty',
// consumer can't acquire mutex to signal 'empty' → DEADLOCK
```

**Correct order: always wait for resource BEFORE locking:**
```c
wait(empty);   // first ensure resource available (may block without holding mutex)
wait(mutex);   // then lock
```

---

## Solution Using Mutex + Condition Variables (Modern approach)

```c
mutex_t lock;
condition_t not_full, not_empty;
int count = 0;  // current items in buffer

// PRODUCER:
void producer() {
    mutex_lock(&lock);
    while (count == N)
        cond_wait(&not_full, &lock);   // releases lock, sleeps until not full

    buffer[in] = produce();
    in = (in + 1) % N;
    count++;

    cond_signal(&not_empty);           // wake up consumer
    mutex_unlock(&lock);
}

// CONSUMER:
void consumer() {
    mutex_lock(&lock);
    while (count == 0)
        cond_wait(&not_empty, &lock);  // releases lock, sleeps until not empty

    int item = buffer[out];
    out = (out + 1) % N;
    count--;

    cond_signal(&not_full);            // wake up producer
    mutex_unlock(&lock);
}
```

Note: `while` loop (not `if`) for condition check — guards against **spurious wakeups**.

---

## Interview Questions & Answers

**Q: What is the Producer-Consumer problem?**
> A classic synchronization problem where a producer generates items into a bounded shared buffer and a consumer removes them. The challenge is: producer must not write to a full buffer, consumer must not read from an empty buffer, and access must be mutually exclusive. Solved with semaphores (empty, full, mutex) or mutex + condition variables.

**Q: Why do we need three semaphores in the Producer-Consumer solution?**
> `empty` counts available empty slots — producer waits when 0. `full` counts available items — consumer waits when 0. `mutex` is a binary semaphore ensuring only one process accesses the buffer at a time. Without `mutex`, two producers could write to the same slot simultaneously.

**Q: What happens if the producer calls wait(mutex) before wait(empty)?**
> Deadlock. If the buffer is full and producer holds mutex, it then waits on `empty`. But the consumer cannot acquire mutex to remove an item and signal `empty`. Both are permanently blocked — classic deadlock.

**Q: What is a spurious wakeup and how do you handle it?**
> A spurious wakeup is when a thread wakes from `cond_wait` without the condition being true (can happen due to OS internals). Guard against it by using a `while` loop to re-check the condition after waking, instead of an `if` statement.

---

*Back to [Topic 2 Index](./README.md)*
