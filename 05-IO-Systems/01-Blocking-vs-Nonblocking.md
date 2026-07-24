#  Blocking vs Non-blocking I/O

---

## The Fundamental Question

When a process requests I/O (read from disk, network, etc.),  
**what does the process do while waiting for the result?**

---

## Blocking I/O

> **The calling process is SUSPENDED (blocked) until the I/O operation completes.**

The process gives up the CPU and enters the **WAITING** state. It only resumes when the data is ready.

```
Process A calls read(fd, buffer, 1024)
    │
    ▼
    [BLOCKED] ← process stops here, CPU given to others
    │
    │  ... disk I/O in progress (OS handles it) ...
    │
    ▼
    [READY → RUNNING] ← OS wakes process when data is ready
    │
    ▼
read() returns, buffer is filled, process continues
```

### Timeline

```
──────────────────────────────────────────────────────→ time
Process: [running][BLOCKED─────────────────][running again]
CPU:     [busy]  [free for other processes ][busy]
                   ↑
              disk I/O happens here (OS + hardware)
```

### Characteristics

| Property | Detail |
|----------|--------|
| Process state during wait | WAITING (blocked) |
| CPU during wait | Given to other processes  |
| Programming model | Simple — linear code flow |
| Result availability | Guaranteed when function returns |
| Use case | Traditional server code, simple scripts |

### Example (Python)

```python
# Blocking I/O — simple and straightforward
f = open("large_file.txt", "r")
data = f.read()          # BLOCKS here until entire file is read
print(data)              # only runs after read is done
```

---

## Non-blocking I/O

> **The call returns IMMEDIATELY, whether or not the data is ready.  
> If data isn't ready, the call returns an error/indicator (e.g., `EAGAIN`).**

The process is never suspended — it keeps running and must check back later.

```
Process A calls read(fd, buffer, 1024)   // fd set to O_NONBLOCK
    │
    ▼
Immediately returns:
  - Data ready?   → returns data 
  - Data NOT ready? → returns -1 with errno = EAGAIN 
    │
    ▼
Process continues running (NOT blocked!)
... does other work ...
... polls again later ...
```

### Timeline

```
──────────────────────────────────────────────────────→ time
Process: [running─────────────────────────────running]
         [try read][EAGAIN][other work][try read][SUCCESS]
CPU:     [always busy with this process]
```

### Characteristics

| Property | Detail |
|----------|--------|
| Process state during wait | RUNNING (never blocks) |
| CPU during wait | Consumed by this process checking / |
| Programming model | Complex — must handle partial/no data |
| Result availability | Must poll or use callbacks |
| Use case | Event loops, high-performance servers (Node.js) |

### Example (Python-like pseudocode)

```python
import fcntl, os
fcntl.fcntl(fd, fcntl.F_SETFL, os.O_NONBLOCK)

while True:
    try:
        data = os.read(fd, 1024)   # returns immediately
        break
    except BlockingIOError:
        # Not ready yet, do other work
        do_other_work()
```

---

## Blocking vs Non-blocking — Side by Side

| Feature | Blocking | Non-blocking |
|---------|---------|-------------|
| **Process suspends?** |  Yes |  No |
| **Returns immediately?** |  No |  Yes (with EAGAIN if not ready) |
| **CPU efficiency** |  Good (CPU free for others) |  May busy-wait |
| **Code complexity** |  Simple |  Complex |
| **Concurrency model** | Thread per connection | Event loop / select/poll/epoll |
| **Example** | Traditional web server (Apache) | Node.js, Nginx |

---

## Real-World Pattern: select() / epoll()

Non-blocking I/O is commonly used with `select()`, `poll()`, or `epoll()` — these let a **single thread watch multiple file descriptors** and react when any of them becomes ready.

```
Single-threaded server using epoll:

epoll_wait([fd1, fd2, fd3, ...])  ← BLOCKS (waits for ANY fd to be ready)
     │
     ▼
fd2 is ready! → handle fd2 → go back to epoll_wait

This is how Nginx serves thousands of connections with very few threads.
```

---

##  Common Interview Trap

> **Blocking ≠ Synchronous**  
> **Non-blocking ≠ Asynchronous**

These are **two separate axes**:

|  | **Synchronous** | **Asynchronous** |
|--|----------------|-----------------|
| **Blocking** | read() waits for data, process blocks | Rare combination |
| **Non-blocking** | read() returns immediately, caller polls | Callback/future when done |

See [02-Synchronous-vs-Asynchronous.md](./02-Synchronous-vs-Asynchronous.md) for full details.

---

##  Interview Questions & Answers

**Q: What is blocking I/O?**
> In blocking I/O, the calling process is suspended (enters WAITING state) until the I/O operation completes. The OS gives the CPU to other processes during the wait. When data is ready, the process is moved back to READY. Simple to program but one thread can only handle one I/O at a time.

**Q: What is non-blocking I/O?**
> In non-blocking I/O, the system call returns immediately regardless of whether data is ready. If data isn't available, it returns an error (e.g., EAGAIN). The process is never suspended — it continues running and must check back (poll) or use a mechanism like epoll to be notified.

**Q: When would you use non-blocking I/O over blocking I/O?**
> Non-blocking I/O is used when a single thread must handle multiple I/O sources simultaneously (e.g., thousands of network connections). Instead of one thread per connection (blocking), you use an event loop (like Node.js or Nginx) that uses epoll to react when any connection has data — highly scalable.

**Q: What is the difference between blocking and non-blocking I/O in terms of process state?**
> Blocking I/O: process moves to WAITING state, CPU is freed for other processes. Non-blocking I/O: process stays in RUNNING state, must actively check for data (potential busy-waiting if not combined with epoll/select).

---

*Next → [Synchronous vs Asynchronous](./02-Synchronous-vs-Asynchronous.md)*
