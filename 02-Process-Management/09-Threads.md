# Threads

---

## Process vs Thread

This is one of the most commonly asked distinctions in SDE interviews.

| | Process | Thread |
|-|---------|--------|
| **Definition** | Program in execution — independent entity | Smallest unit of execution within a process |
| **Address space** | Own separate address space | Shares address space with other threads in same process |
| **Resources** | Own memory, file handles, PCB | Shares heap, code, data, open files of the process |
| **Creation cost** | Expensive (fork = copy address space) | Cheap (just a new stack + registers) |
| **Context switch** | Expensive (TLB flush, page table switch) | Cheap (same address space, no TLB flush) |
| **Communication** | IPC needed (pipes, sockets, shared mem) | Directly via shared memory (but needs synchronization) |
| **Crash impact** | Only that process crashes | One thread crash can kill the whole process |
| **Example** | Chrome browser process | Each Chrome tab's rendering thread |

```
Process (one address space):
┌──────────────────────────────────────────────┐
│  Code segment (shared)                       │
│  Data segment (shared globals)               │
│  Heap (shared — malloc/free)                 │
├──────────────┬──────────────┬────────────────┤
│ Thread 1     │ Thread 2     │ Thread 3       │
│ Stack (own)  │ Stack (own)  │ Stack (own)    │
│ Registers    │ Registers    │ Registers      │
│ PC (own)     │ PC (own)     │ PC (own)       │
└──────────────┴──────────────┴────────────────┘
```

Each thread has its own: **stack, program counter, registers, thread ID**  
Threads share: **code, data, heap, open file descriptors, signals**

---

## Thread Lifecycle

Mirrors the process lifecycle closely:

```
NEW ──→ READY ──→ RUNNING ──→ WAITING ──→ READY ──→ ...
                      └──────────────────────────────→ TERMINATED
```

| State | Trigger |
|-------|---------|
| New | `pthread_create()` / `new Thread()` called |
| Ready | Thread created, waiting for CPU |
| Running | Scheduler assigns CPU |
| Waiting | I/O, `sleep()`, waiting for mutex/semaphore |
| Terminated | `pthread_exit()` / `return` from thread function |

---

## Multithreading Models

How user threads map to kernel threads:

### 1. Many-to-One
```
User Threads:  T1  T2  T3  T4  T5
                └───┴───┴───┴───┘
                       │
Kernel Thread:         K1
```
- All user threads map to ONE kernel thread
- If one blocks on I/O → ALL block
- No true parallelism even on multi-core
- Example: Green threads (old Java, early Python)

---

### 2. One-to-One
```
User Threads:  T1   T2   T3   T4
                │    │    │    │
Kernel Threads: K1   K2   K3   K4
```
- Each user thread has its own kernel thread
- True parallelism on multi-core
- Thread creation is heavier (kernel involvement)
- Example: Linux pthreads, Windows threads

---

### 3. Many-to-Many
```
User Threads:  T1  T2  T3  T4  T5  T6
                └──┬───┴─┐  └──┬───┘
Kernel Threads:    K1    K2    K3
```
- Multiple user threads multiplexed onto fewer kernel threads
- Best flexibility — no blocking, parallelism possible
- Complex to implement
- Example: Solaris, some Go runtime implementations

---

## Advantages of Threads over Processes

| Advantage | Detail |
|-----------|--------|
| Faster creation | No need to copy address space |
| Faster communication | Shared memory — no IPC overhead |
| Faster context switch | Same address space — no TLB flush |
| Resource efficiency | All threads share memory — less RAM used |
| Responsiveness | One thread does I/O, others keep running |
| Parallelism | Multiple threads on multiple CPU cores simultaneously |

---

## User Threads vs Kernel Threads

| | User-Level Threads | Kernel-Level Threads |
|-|-------------------|---------------------|
| **Managed by** | Thread library (user space) | OS kernel |
| **Context switch** | Fast (no kernel involvement) | Slower (syscall needed) |
| **I/O blocking** | Blocks entire process | Only that thread blocks |
| **Parallelism** | Not on multi-core | True parallelism |
| **Example** | POSIX user threads, early Java | Linux pthreads, Windows threads |

---

## Thread vs Process — When to Use Which?

```
Use THREADS when:
  - Tasks share data frequently (same address space → easy sharing)
  - Low overhead communication needed
  - Need high performance on multi-core (parallel computation)
  - Example: Web server handling multiple requests

Use PROCESSES when:
  - Tasks need strong isolation (crash in one shouldn't affect others)
  - Running different programs
  - Security boundaries required
  - Example: Browser isolating each tab (Chrome), microservices
```

---

## POSIX Threads (pthreads) — Quick Reference

```c
#include <pthread.h>

void* thread_func(void* arg) {
    int id = *(int*)arg;
    printf("Thread %d running\n", id);
    return NULL;
}

int main() {
    pthread_t t1, t2;
    int id1 = 1, id2 = 2;

    pthread_create(&t1, NULL, thread_func, &id1);  // create thread
    pthread_create(&t2, NULL, thread_func, &id2);

    pthread_join(t1, NULL);   // wait for t1 to finish
    pthread_join(t2, NULL);   // wait for t2 to finish
    return 0;
}
```

---

## Interview Questions & Answers

**Q: What is the difference between a process and a thread?**
> A process is an independent program in execution with its own address space and OS resources. A thread is a lightweight unit of execution within a process — threads share the process's code, data, heap, and open files but have their own stack, program counter, and registers. Threads are cheaper to create and switch between, and communicate via shared memory instead of IPC.

**Q: What do threads share and what is private?**
> Shared: code segment, data segment (globals), heap memory, open file descriptors, process ID. Private: stack, program counter, registers, thread ID, errno.

**Q: Why is thread context switching faster than process context switching?**
> Threads share the same address space, so the page table doesn't change and the TLB doesn't need to be flushed. Only the stack pointer, program counter, and registers are swapped — much less work than a full process context switch.

**Q: What is a race condition in threads?**
> A race condition occurs when two threads access shared data simultaneously without synchronization, and the result depends on the order of execution. For example, two threads both incrementing a shared counter without a mutex can produce incorrect results because increment is not atomic (it's load, add, store at the hardware level).

**Q: What is the difference between user-level threads and kernel-level threads?**
> User-level threads are managed by a thread library in user space — fast to create and switch but if one blocks on I/O, all threads in the process block. Kernel-level threads are managed by the OS — if one blocks, others continue; true parallelism on multi-core; but creation and switching involve system calls (heavier).

---

*Back to [Topic 2 Index](./README.md)*
