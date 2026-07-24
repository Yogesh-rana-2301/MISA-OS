# Inter-Process Communication (IPC)

---

## Why IPC?

Processes have **separate address spaces** — they cannot directly read each other's memory (by design, for protection). IPC mechanisms allow processes to exchange data and coordinate.

```
Process A               Process B
[private memory]        [private memory]
       │                       │
       └───── IPC mechanism ───┘
              (controlled channel)
```

---

## IPC Mechanisms Overview

| Mechanism | Direction | Speed | Persistence | Use Case |
|-----------|-----------|-------|-------------|---------|
| Shared Memory | Bidirectional | Very fast | No | High-speed data sharing |
| Message Queue | Bidirectional | Fast | Yes (in kernel) | Async messaging |
| Pipe (anonymous) | Unidirectional | Fast | No | Parent-child communication |
| Named Pipe (FIFO) | Unidirectional | Fast | No | Unrelated processes |
| Socket | Bidirectional | Fast/slow | No | Network or local comm |
| Signal | One-way notification | Fast | No | Event notification |

---

## 1. Shared Memory

> Two (or more) processes map the **same physical memory region** into their address spaces. Data written by one is immediately visible to the other.

```
Process A address space:        Process B address space:
┌──────────────────┐            ┌──────────────────┐
│  private memory  │            │  private memory  │
├──────────────────┤            ├──────────────────┤
│  shared region   │←──────────▶│  shared region   │
│  (same physical) │            │  (same physical) │
└──────────────────┘            └──────────────────┘
```

- Fastest IPC — no data copying, just memory reads/writes
- Requires synchronization (mutex/semaphore) to avoid race conditions
- System calls: `shmget()`, `shmat()`, `shmdt()`, `shmctl()` (POSIX: `shm_open()`)

```
Use case: Video processing pipeline — one process captures frames,
          another processes them. Share a frame buffer in shared memory.
```

---

## 2. Message Queue

> Processes communicate by sending and receiving **discrete messages** via a kernel-managed queue.

```
Process A ──[send msg]──→ [ msg3 | msg2 | msg1 ] ──[receive]──→ Process B
                            kernel message queue
```

- Messages persist in kernel until received (unlike pipes)
- Supports multiple senders and receivers
- FIFO ordering by default; can prioritize messages
- System calls: `msgget()`, `msgsnd()`, `msgrcv()`, `msgctl()`

```
Use case: Task queue — web server dispatches jobs to worker processes.
          Each job is a message; workers pick them up independently.
```

---

## 3. Pipe (Anonymous Pipe)

> A unidirectional byte stream connecting two processes. Data written to one end is read from the other.

```
Process A (write end) ──[bytes]──→ [PIPE BUFFER] ──→ Process B (read end)
```

- Unidirectional only (for bidirectional, use two pipes)
- Only works between **related processes** (parent-child via fork)
- Data is a raw byte stream — no message boundaries
- System call: `pipe(int fd[2])` — `fd[0]` = read end, `fd[1]` = write end

```c
int fd[2];
pipe(fd);
if (fork() == 0) {
    // Child reads
    close(fd[1]);
    read(fd[0], buffer, sizeof(buffer));
} else {
    // Parent writes
    close(fd[0]);
    write(fd[1], "hello", 5);
}
```

```
Use case: Shell pipelines — ls | grep ".txt" | wc -l
          ls stdout → pipe → grep stdin → pipe → wc stdin
```

---

## 4. Named Pipe (FIFO)

> Like a pipe, but has a **name in the filesystem** — can connect **unrelated processes**.

```
Process A ──write──→ /tmp/mypipe ──read──→ Process B
                    (filesystem entry)
```

- Persists in the filesystem until explicitly deleted
- Unidirectional
- System call: `mkfifo("/tmp/mypipe", 0666)`

```
Use case: A logging daemon reads from a named pipe. Any process can
          write log messages to it without knowing the daemon's PID.
```

---

## 5. Socket

> An endpoint for communication — can work across a network or locally on the same machine (Unix domain sockets).

```
                    Network Socket:
Process A ──→ TCP/IP stack ──→ Network ──→ TCP/IP stack ──→ Process B

                    Unix Domain Socket (local only):
Process A ──→ /tmp/app.sock ──→ Process B   (faster, no network stack)
```

- **Bidirectional** — both sides can send and receive
- Can be TCP (reliable, ordered), UDP (unreliable, fast), or Unix domain sockets
- Most flexible IPC — works locally and across machines

```
Use case: Web server ↔ client communication (HTTP over TCP socket).
          PostgreSQL uses Unix domain sockets for local client connections.
```

---

## 6. Signals

> A signal is an **asynchronous notification** sent to a process to notify it of an event.

```
Process B ──kill(pid_A, SIGUSR1)──→ Process A
                                          │
                           Signal handler runs in A
```

### Common Signals

| Signal | Number | Default Action | Meaning |
|--------|--------|---------------|---------|
| SIGTERM | 15 | Terminate | Polite termination request |
| SIGKILL | 9 | Terminate | Force kill (cannot be caught!) |
| SIGINT | 2 | Terminate | Ctrl+C from keyboard |
| SIGSEGV | 11 | Core dump | Segmentation fault |
| SIGCHLD | 17 | Ignore | Child process terminated |
| SIGUSR1/2 | 10/12 | Terminate | User-defined signals |
| SIGALRM | 14 | Terminate | Timer expired |

- Not for transferring data — just notifications
- Only one bit of information per signal (the signal number)
- Handlers can be customized (except SIGKILL and SIGSTOP)

```c
#include <signal.h>

void handler(int sig) {
    printf("Received signal %d\n", sig);
}

signal(SIGUSR1, handler);   // register handler
```

---

## IPC Comparison — When to Use What?

| Scenario | Best IPC |
|----------|---------|
| High-speed data sharing between related processes | Shared Memory |
| Async task queue, multiple producers/consumers | Message Queue |
| Parent-child data streaming | Pipe |
| Unrelated local processes, file-based | Named Pipe |
| Cross-machine communication or flexible local | Socket |
| Simple event notification | Signal |

---

## Interview Questions & Answers

**Q: What is IPC? Why do processes need it?**
> IPC (Inter-Process Communication) refers to mechanisms that allow processes to exchange data and coordinate. Since processes have separate address spaces (by OS design for protection), they cannot directly access each other's memory and must use controlled channels: shared memory, pipes, message queues, sockets, or signals.

**Q: What is the fastest IPC mechanism and why?**
> Shared memory is the fastest because data is written directly to a shared physical memory region — there's no copying. The OS doesn't handle each data transfer; processes read/write memory directly. All other mechanisms involve at least one data copy through the kernel.

**Q: What is the difference between a pipe and a named pipe?**
> An anonymous pipe only works between related processes (parent-child, created before fork). It has no filesystem presence. A named pipe (FIFO) has a name in the filesystem and can connect any two unrelated processes. Both are unidirectional byte streams.

**Q: What is the difference between a message queue and a pipe?**
> A pipe is a raw byte stream with no message boundaries — the receiver must know how to parse the data. A message queue delivers discrete, structured messages with preserved boundaries. Message queues also persist in the kernel (messages survive even if the sender dies), while pipes need both ends open.

**Q: What signals cannot be caught or ignored?**
> SIGKILL (9) and SIGSTOP cannot be caught, blocked, or ignored — they are always handled by the OS kernel directly. This ensures the system always has a way to forcefully terminate or pause a misbehaving process.

---

*Back to [Topic 2 Index](./README.md)*
