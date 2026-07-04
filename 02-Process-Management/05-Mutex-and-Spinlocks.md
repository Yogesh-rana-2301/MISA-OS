# 🔒 Mutex & Spinlocks

---

## Mutex (Mutual Exclusion Lock)

A **mutex** is a locking mechanism that ensures **only one process/thread** can access a shared resource at a time.

> **Analogy**: A bathroom with a single key hanging outside. One person takes the key (locks), uses the bathroom (CS), hangs the key back (unlocks). Others wait outside.

### Mutex Operations

```c
mutex_t m;          // mutex object

// Acquire lock:
mutex_lock(&m);
//  ─── CRITICAL SECTION ───
//  Access shared resource
//  ─────────────────────
// Release lock:
mutex_unlock(&m);
```

### How Mutex Works Internally

```
mutex_lock():
  if (lock is free):
      set lock = held (atomically using test_and_set or CAS)
      return  // entered CS
  else:
      BLOCK this process (add to wait queue)
      // OS puts process to sleep — no CPU wasted!

mutex_unlock():
  set lock = free
  if (wait queue is not empty):
      wake up one waiting process
```

### Key Properties

| Property | Detail |
|----------|--------|
| **Blocking** | ✅ Process sleeps while waiting (no CPU waste) |
| **Owner** | Mutex has an owner — only the owner can unlock |
| **Use case** | Protecting shared data structures |
| **Overhead** | System call overhead for sleep/wake |

---

## Spinlock

A **spinlock** is a lock where the waiting process **continuously polls (spins)** the lock until it becomes free.

```c
// Spinlock acquire:
while (test_and_set(&lock) == LOCKED) {
    // SPIN: keep checking!
    // CPU is actively running — no sleep
}
// CS entered

// Spinlock release:
lock = FREE;
```

### What "Spinning" Looks Like

```
Process B wants the lock:
─────────────────────────────────────────────────────────
Time:  1  2  3  4  5  6  7  8  9  10  11  12  13
P_B:  spin spin spin spin spin spin spin spin spin [LOCK] [CS]
P_A:  [──────────────────── CS ──────────────────] [UNLOCK]
```

P_B burns CPU cycles doing nothing useful while spinning.

### Key Properties

| Property | Detail |
|----------|--------|
| **Blocking** | ❌ No — process stays on CPU (busy-waiting) |
| **CPU waste** | Wastes CPU during wait |
| **Fast** | ✅ No context switch overhead |
| **Best for** | Very short CS durations |

---

## Mutex vs Spinlock — Side-by-Side

| Feature | Mutex | Spinlock |
|---------|-------|----------|
| **Waiting behavior** | Sleeps (blocks) | Busy-waits (spins) |
| **CPU while waiting** | Freed (other processes can run) | Consumed (spinning) |
| **Context switch** | Yes (on block and wake) | No |
| **Best for** | Long critical sections | Very short critical sections |
| **Overhead** | Higher (OS involvement) | Lower per iteration |
| **Multi-core friendly** | Always | ✅ Yes, especially on multi-core |
| **Single-core** | ✅ Good | ❌ Bad (spinlock = deadlock if holder preempted) |

---

## When to Use Which?

```
Critical section duration:

    Short (μs)              Long (ms+)
        │                      │
        ▼                      ▼
   SPINLOCK               MUTEX (sleep)
   
   (Context switch would    (Sleeping frees CPU
    take longer than CS)     for other work)
```

### Real-World Usage

| System | Uses |
|--------|------|
| **OS kernel (multi-core)** | Spinlocks (for very short kernel data structure locks) |
| **User-space applications** | Mutex (threads protecting shared data) |
| **Database connections** | Mutex |
| **Interrupt handlers** | Spinlocks (can't sleep in interrupt context) |

---

## 🔧 test_and_set — The Atomic Instruction Behind Both

```c
// Atomic (hardware-guaranteed): read AND set in ONE uninterruptible step
bool test_and_set(bool *lock) {
    bool old = *lock;
    *lock = true;
    return old;
}

// Spinlock using test_and_set:
while (test_and_set(&lock) == true);  // spin while lock was held
// CS here
lock = false;
```

The **atomicity** is the key — the CPU guarantees no other core can interfere between the read and write.

---

## ⚠️ Common Pitfalls

| Problem | Description |
|---------|-------------|
| **Forgetting to unlock** | Lock held forever → other threads starve |
| **Double lock** | Locking a mutex you already hold → self-deadlock |
| **Lock ordering** | Always acquire multiple locks in the same order to avoid deadlock |
| **Spinlock on single core** | If the holder gets preempted, spinner wastes entire CPU |

---

## 🎯 Interview Questions & Answers

**Q: What is a mutex?**
> A mutex (mutual exclusion lock) is a synchronization primitive that ensures only one thread/process can enter a critical section at a time. A thread that can't acquire the mutex blocks (sleeps), freeing the CPU for other work.

**Q: What is a spinlock?**
> A spinlock is a lock where the waiting thread continuously loops (spins), checking if the lock is free. It doesn't sleep, so it wastes CPU but avoids context switch overhead. Best for very short critical sections on multi-core systems.

**Q: When would you use a spinlock over a mutex?**
> Use a spinlock when the critical section is very short (microseconds) — shorter than the time to block/wake a thread. Common in OS kernels and interrupt handlers. Use a mutex when the CS is longer, so the CPU can do useful work while waiting.

**Q: What is the difference between a mutex and a semaphore?**
> A mutex is binary (locked/unlocked), has an owner (only the locker can unlock it), and is used for mutual exclusion. A semaphore is a counter, has no owner, and can be used for both mutual exclusion (binary semaphore) and signaling between processes (counting semaphore). See [Semaphores](./06-Semaphores.md).

**Q: What is test_and_set?**
> An atomic hardware instruction that reads the current value of a variable and sets it to true in a single uninterruptible operation. It's the hardware primitive used to implement spinlocks and mutexes correctly, preventing race conditions on the lock variable itself.

---

*← [Process Synchronization](./04-Process-Synchronization.md) | Next → [Semaphores](./06-Semaphores.md)*
