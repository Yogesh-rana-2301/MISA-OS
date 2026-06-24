# ⚡ Multiprogramming vs Multitasking

---

## 🔑 Core Idea

Both concepts aim to **keep the CPU busy** and avoid wasted time. But they solve different problems.

---

## 📖 Definitions

### Multiprogramming

> **Multiple programs are loaded in memory simultaneously**, and the CPU switches to another program when the current one is waiting (e.g., for I/O).

**Key property**: CPU switches only when a process **blocks** (waits for I/O).

```
Without Multiprogramming (single program):
─────────────────────────────────────────
| CPU runs P1 | CPU IDLE (P1 waits I/O) | CPU runs P1 again |
                ↑
          Wasted CPU time!

With Multiprogramming:
─────────────────────────────────────────
| CPU runs P1 | CPU runs P2 (P1 waits) | CPU runs P1 again |
                ↑
          CPU never idle!
```

---

### Multitasking (Time-Sharing)

> **Multiple tasks share the CPU**, each getting a **time slice (quantum)**. The CPU switches between processes even if they are not waiting for I/O.

**Key property**: CPU switches based on a **timer** (preemptive).

```
Multitasking with time quantum = 20ms:
─────────────────────────────────────────────────
| P1 (20ms) | P2 (20ms) | P3 (20ms) | P1 (20ms) |
```

---

## ⚔️ Key Differences

| Feature | Multiprogramming | Multitasking |
|---------|----------------|-------------|
| **Goal** | Maximize CPU utilization | Enable fast user response |
| **Switching trigger** | Process blocks (I/O wait) | Time quantum expires OR block |
| **Switching type** | Non-preemptive (cooperative) | Preemptive (timer-based) |
| **User interaction** | Minimal (batch-like) | High (interactive) |
| **Number of users** | Usually one | Multiple users |
| **Response time** | Slower | Fast (feels real-time) |
| **CPU idle time** | Reduced | Minimized further |
| **Example OS** | Early UNIX (batch style) | Modern Linux, Windows |

---

## 📈 CPU Utilization Concept

**CPU Utilization** = percentage of time the CPU is doing useful work.

### Formula (Rough Estimate)

If a process spends fraction `p` of time waiting for I/O:

```
CPU Utilization ≈ 1 - p^n
  where n = number of processes in memory
```

### Example

| Processes (n) | p = 0.8 (80% I/O wait) | CPU Utilization |
|--------------|------------------------|-----------------|
| 1 | 0.8¹ | 20% |
| 2 | 0.8² | 36% |
| 4 | 0.8⁴ | 59% |
| 8 | 0.8⁸ | 83% |
| 16 | 0.8¹⁶ | 97% |

> **Takeaway**: More processes in memory → Higher CPU utilization → Less wasted time.

---

## 🧵 Related Concepts

### Multiprocessing

> A system with **multiple CPUs (cores)** running processes truly in **parallel** (not just switching).

```
Multitasking (1 CPU):
Core 1: P1──P2──P3──P1──P2

Multiprocessing (2 CPUs):
Core 1: P1──────P1──────
Core 2: P2──────P2──────   ← TRUE parallel execution
```

### Quick Comparison: All Three

| | Multiprogramming | Multitasking | Multiprocessing |
|-|-----------------|-------------|----------------|
| CPUs | 1 | 1 | Multiple |
| Parallelism | Fake (interleaved) | Fake (interleaved) | True |
| Switch Trigger | I/O block | Timer / I/O | — |
| Goal | CPU utilization | Fast response | Max throughput |

---

## 🎯 Interview Questions & Answers

**Q: What is the difference between multiprogramming and multitasking?**
> Multiprogramming keeps multiple programs in memory and switches the CPU when one blocks on I/O — it's about CPU utilization. Multitasking uses time slices to switch between processes rapidly, enabling interactive responsiveness.

**Q: How does multiprogramming improve CPU utilization?**
> When one process waits for I/O, the CPU would otherwise be idle. With multiprogramming, the OS switches to another ready process. Using the formula `1 - p^n`, with 8 processes each spending 80% time on I/O, CPU utilization jumps from 20% to 83%.

**Q: Is modern Linux/Windows multiprogrammed or multitasked?**
> Both. Modern OSes are **multitasking** (preemptive time-slicing) AND **multiprogrammed** (multiple processes in memory). They combine both techniques.

**Q: What is the difference between multitasking and multiprocessing?**
> Multitasking = 1 CPU switching between tasks quickly (illusion of parallelism). Multiprocessing = multiple CPUs running tasks truly in parallel simultaneously.

---

*← [Types of OS](./02-Types-of-OS.md) | Next → [Kernel and User Mode](./04-Kernel-and-User-Mode.md)*
