#  Synchronous vs Asynchronous I/O

---

## The Fundamental Question

**Does the caller wait for the I/O to COMPLETE before continuing?**

- **Synchronous**: Caller waits (or polls) until the operation finishes.
- **Asynchronous**: Caller fires the request and moves on; gets notified when done.

---

## Synchronous I/O

> **The caller waits (or actively polls) until the I/O operation is fully complete.**  
> The caller and the I/O operation are in "sync" — they progress together.

```
Caller:   [issue request] ─── WAIT ─── [receive result] [continue]
I/O:                          [════════════ executing ══]
```

The caller's next instruction only runs **after** the I/O completes.

### Two forms of Synchronous I/O

| | Blocking Sync | Non-blocking Sync |
|-|--------------|------------------|
| **Process state** | WAITING (blocked) | RUNNING (polling) |
| **CPU** | Free for others | Consumed polling |
| **Returns** | After completion | Immediately (EAGAIN if not ready) |
| **Example** | `read()` on regular file | `read()` on O_NONBLOCK fd, caller loops |

```
Blocking Synchronous (most common "sync"):
  data = read(fd, buf, n)   // process blocks, waits, returns when done

Non-blocking Synchronous (polling):
  while True:
      result = read(fd, buf, n)   // returns EAGAIN immediately if not ready
      if result != EAGAIN: break  // caller keeps checking
```

---

## Asynchronous I/O

> **The caller issues the I/O request and immediately continues doing other work.  
> The OS/kernel notifies the caller (via callback, signal, or event) when I/O completes.**

```
Caller:   [issue request] ────────────────────────── [callback runs!] [continue]
               │                                            ↑
               │                                     OS notifies caller
I/O:           [══════════════════ executing ═══════]
```

The caller doesn't wait at all — it gets notified asynchronously.

### How Notification Works

| Mechanism | How |
|-----------|-----|
| **Callback function** | OS calls a registered function when done |
| **Signal** | OS sends a signal (e.g., SIGIO) to the process |
| **Future/Promise** | Caller gets a handle; queries it later |
| **Event queue** | Completion event placed in a queue (aio, io_uring) |

### Example (Conceptual)

```c
// Asynchronous I/O using POSIX AIO:
struct aiocb cb;
cb.aio_fildes = fd;
cb.aio_buf = buffer;
cb.aio_nbytes = 1024;

aio_read(&cb);         // Submit request — returns IMMEDIATELY
do_other_work();       // Caller runs other code while I/O happens

// Check later:
while (aio_error(&cb) == EINPROGRESS) {
    do_more_work();
}
result = aio_return(&cb);  // Get result when done
```

---

## The 2×2 Matrix — Combining Both Concepts

```
                    SYNCHRONOUS          ASYNCHRONOUS
                  ┌─────────────────┬──────────────────────┐
   BLOCKING       │ read() — blocks │ (rare/unusual)       │
                  │ until data ready│                      │
                  ├─────────────────┼──────────────────────┤
   NON-BLOCKING   │ O_NONBLOCK +    │ aio_read(),          │
                  │ polling loop    │ io_uring, callbacks  │
                  └─────────────────┴──────────────────────┘
```

| Combination | Description | Example |
|-------------|-------------|---------|
| **Blocking + Sync** | Most common. Process waits, returns when done | `read()`, `fread()` |
| **Non-blocking + Sync** | Process polls in a loop, never suspends | O_NONBLOCK + spin |
| **Non-blocking + Async** | Fire and forget, notified via callback/event | Node.js, `io_uring`, `aio_read()` |
| **Blocking + Async** | Uncommon — OS blocks internally but caller continues | (theoretical) |

---

## Real Examples

### Synchronous (Blocking) — Traditional Server

```python
# Traditional blocking server — one thread per client
while True:
    conn = server.accept()       # blocks until a client connects
    data = conn.recv(1024)       # blocks until client sends data
    conn.send(response)          # blocks until data sent
```

Problem: 1000 clients → 1000 threads → massive overhead.

---

### Asynchronous — Node.js Style

```javascript
// Asynchronous I/O — Node.js
fs.readFile('data.txt', (err, data) => {
    // This callback runs LATER when file is read
    console.log(data);
});

// This runs IMMEDIATELY after issuing read — doesn't wait!
console.log("File read requested, continuing...");
```

Output order:
```
File read requested, continuing...   ← runs first
(file contents)                      ← runs later via callback
```

---

## Sync vs Async — Practical Comparison

| Aspect | Synchronous | Asynchronous |
|--------|------------|-------------|
| **Caller waits?** |  Yes |  No |
| **Notification** | Result returned directly | Callback / event / signal |
| **Code complexity** |  Simple, linear |  Complex (callbacks, promises) |
| **Concurrency** | Need threads for scale | Single thread handles many I/Os |
| **Latency** | Higher (waits for each) | Lower (multiple in-flight) |
| **Use case** | Simple programs, scripts | High-performance servers, GUIs |
| **Examples** | `read()`, `fgets()` | `aio_read()`, Node.js, `io_uring` |

---

##  Interview Questions & Answers

**Q: What is the difference between synchronous and asynchronous I/O?**
> Synchronous I/O: the caller waits (or polls) until the I/O operation completes before moving to the next instruction. Asynchronous I/O: the caller fires the request and immediately continues; the OS notifies the caller (via callback, signal, or event) when the operation finishes.

**Q: What is the difference between blocking and asynchronous I/O?**
> Blocking is about the process state — does the process suspend? Asynchronous is about when the result is delivered — does the caller wait for it? Asynchronous I/O is inherently non-blocking (caller gets the result via notification later), but non-blocking I/O doesn't automatically mean asynchronous (the caller might poll synchronously).

**Q: How does Node.js handle thousands of concurrent connections on a single thread?**
> Node.js uses asynchronous, non-blocking I/O with an event loop. I/O operations are issued to the OS (via libuv) and the event loop continues processing other events. When I/O completes, the OS notifies Node.js, which queues the callback for execution. No thread-per-connection needed.

**Q: Give a real-world example of asynchronous I/O.**
> When you click "Download" in a browser, the browser issues an async network request and returns control to the UI immediately — you can still scroll and click other things. When the download completes, a notification/callback updates the progress bar. The UI thread never blocked.

---

*← [Blocking vs Non-blocking](./01-Blocking-vs-Nonblocking.md) | Next → [Spooling](./03-Spooling.md)*
