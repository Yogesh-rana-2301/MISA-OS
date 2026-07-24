# Classic Synchronization Problem 2 — Reader-Writer Problem

---

## Problem Statement

A shared data structure (database, file) is accessed by:
- **Readers** — only read the data (multiple can read simultaneously — safe)
- **Writers** — modify the data (only one at a time, no concurrent readers)

**Rules:**
1. Multiple readers can read **simultaneously** — no conflict.
2. Only **one writer** can write at a time.
3. When a writer is writing, **no reader can read**.

```
Database:
  Readers:  R1, R2, R3 reading at same time → OK
  Writer W1 writing → no one else can access → OK
  W1 writing + R1 reading at same time → NOT OK (data corruption)
```

---

## Variants

| Variant | Priority | Problem |
|---------|----------|---------|
| **Readers-priority** | Readers preferred | Writers can starve |
| **Writers-priority** | Writers preferred | Readers can starve |
| **Fair (no starvation)** | Neither favored | More complex |

---

## Solution 1: Readers-Priority

Readers are given priority — as long as readers are present, a waiting writer must wait.

```c
semaphore mutex = 1;     // protects read_count
semaphore db    = 1;     // controls access to database
int read_count  = 0;     // how many readers currently reading

// READER:
void reader() {
    wait(mutex);             // lock read_count
    read_count++;
    if (read_count == 1)     // first reader locks out writers
        wait(db);
    signal(mutex);           // unlock read_count

    // ── READ DATABASE ──

    wait(mutex);             // lock read_count
    read_count--;
    if (read_count == 0)     // last reader unlocks for writers
        signal(db);
    signal(mutex);           // unlock read_count
}

// WRITER:
void writer() {
    wait(db);                // exclusive access to database
    // ── WRITE DATABASE ──
    signal(db);
}
```

### Trace

```
R1 arrives: read_count=1, wait(db) → db=0 (writer blocked)
R2 arrives: read_count=2, db already locked by R1's group
R3 arrives: read_count=3

W1 arrives: wait(db) → db=0 → W1 BLOCKED (readers have it)

R1 finishes: read_count=2
R2 finishes: read_count=1
R3 finishes: read_count=0, signal(db) → db=1, W1 WAKES UP

W1 runs: db=0 (exclusive write)
```

### Problem: Writer Starvation

If readers keep arriving, `read_count` never reaches 0, and the writer **starves forever**.

---

## Solution 2: Writers-Priority

Add a mechanism to queue waiting writers, blocking new readers when a writer is waiting.

```c
semaphore mutex1  = 1;   // protects read_count
semaphore mutex2  = 1;   // protects write_count
semaphore db      = 1;   // database access
semaphore reader_gate = 1; // blocks new readers when writer waiting

int read_count  = 0;
int write_count = 0;

// READER:
void reader() {
    wait(reader_gate);       // may be blocked by waiting writer
    wait(mutex1);
    read_count++;
    if (read_count == 1) wait(db);
    signal(mutex1);
    signal(reader_gate);

    // ── READ ──

    wait(mutex1);
    read_count--;
    if (read_count == 0) signal(db);
    signal(mutex1);
}

// WRITER:
void writer() {
    wait(mutex2);
    write_count++;
    if (write_count == 1) wait(reader_gate); // first writer blocks new readers
    signal(mutex2);

    wait(db);
    // ── WRITE ──
    signal(db);

    wait(mutex2);
    write_count--;
    if (write_count == 0) signal(reader_gate); // last writer releases readers
    signal(mutex2);
}
```

---

## Summary

| | Readers-Priority | Writers-Priority |
|-|-----------------|-----------------|
| **Concurrent readers** | Yes | Yes |
| **Writer exclusion** | Yes | Yes |
| **Reader starvation** | No | Possible |
| **Writer starvation** | Possible | No |
| **Complexity** | Simple | More complex |

---

## Interview Questions & Answers

**Q: What is the Reader-Writer problem?**
> A synchronization problem where a shared resource can be read by multiple readers simultaneously, but writing requires exclusive access — no concurrent readers or writers. The challenge is allowing maximum concurrency for readers while ensuring writers get consistent exclusive access.

**Q: Why can multiple readers access the database simultaneously?**
> Read-only operations don't modify data, so there is no risk of data inconsistency if multiple readers access it at the same time. Writers modify data, so any concurrent access (read or write) during a write could see partially updated, inconsistent data.

**Q: What is the difference between readers-priority and writers-priority solutions?**
> In readers-priority, readers block writers as long as any reader is active — writers starve if readers keep arriving. In writers-priority, when a writer arrives, new readers are blocked and must wait for the writer — readers starve if writers keep arriving. A fair solution uses a queue to prevent starvation of either.

**Q: What semaphores does the readers-priority solution use and why?**
> `mutex` (binary): protects the `read_count` variable itself from race conditions. `db` (binary): controls exclusive database access. The first reader acquires `db`, blocking writers; the last reader releases it, allowing writers in. Writers directly acquire/release `db` for exclusive access.

---

*Back to [Topic 2 Index](./README.md)*
