#  CPU Scheduling Algorithms

---

## Why Scheduling?

When multiple processes are **Ready**, the OS must decide which one runs next on the CPU.  
The **CPU Scheduler** makes this decision.

---

##  Key Metrics (Know These Cold)

| Term | Formula | Meaning |
|------|---------|---------|
| **Arrival Time (AT)** | — | When process enters Ready queue |
| **Burst Time (BT)** | — | CPU time the process needs |
| **Completion Time (CT)** | — | When process finishes |
| **Turnaround Time (TAT)** | CT − AT | Total time from arrival to finish |
| **Waiting Time (WT)** | TAT − BT | Time spent waiting in queue |
| **Response Time** | First CPU − AT | Time until first CPU response |

> **Average WT and TAT** are the most common metrics in interview problems.

---

## 1.  FCFS — First Come First Served

### How It Works
Processes are executed in the **order they arrive**. Non-preemptive.

### Example

| Process | AT | BT |
|---------|----|----|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 4 |

**Gantt Chart:**
```
| P1  P1  P1  P1  P1 | P2  P2  P2 | P3  P3  P3  P3 |
0                   5           8               12
```

**Calculations:**

| Process | AT | BT | CT | TAT (CT−AT) | WT (TAT−BT) |
|---------|----|----|----|-----------  |-------------|
| P1 | 0 | 5 | 5 | 5 | 0 |
| P2 | 1 | 3 | 8 | 7 | 4 |
| P3 | 2 | 4 | 12 | 10 | 6 |

**Avg WT = (0+4+6)/3 = 3.33**  
**Avg TAT = (5+7+10)/3 = 7.33**

### Properties

| Property | Detail |
|----------|--------|
| Preemptive? |  No |
| Simple? |  Very easy to implement |
| Convoy Effect |  Yes — short jobs wait behind long ones |
| Starvation |  No — everyone eventually runs |

### Convoy Effect
```
Long job P1 (BT=100) arrives at t=0
Short jobs P2, P3, P4 (BT=1 each) arrive at t=1
→ P2, P3, P4 all wait 100ms for P1 to finish!
```

---

## 2.  SJF — Shortest Job First

### How It Works
The process with the **shortest Burst Time** runs next.  
Gives **minimum average waiting time** (provably optimal for non-preemptive).

### Two Variants

| Variant | Description |
|---------|-------------|
| **Non-Preemptive SJF** | Once a process starts, it runs to completion |
| **Preemptive SJF (SRTF)** | New arrival can preempt if it has shorter remaining time |

---

### Non-Preemptive SJF Example

| Process | AT | BT |
|---------|----|----|
| P1 | 0 | 6 |
| P2 | 1 | 4 |
| P3 | 2 | 2 |
| P4 | 3 | 5 |

**Gantt Chart:**
```
| P1(6) | P3(2) | P2(4) | P4(5) |
0       6       8      12      17
```
*(At t=6, ready: P2(4), P3(2), P4(5) → pick P3 shortest)*

| Process | AT | BT | CT | TAT | WT |
|---------|----|----|----|----|-----|
| P1 | 0 | 6 | 6 | 6 | 0 |
| P2 | 1 | 4 | 12 | 11 | 7 |
| P3 | 2 | 2 | 8 | 6 | 4 |
| P4 | 3 | 5 | 17 | 14 | 9 |

**Avg WT = (0+7+4+9)/4 = 5.0**

---

### Preemptive SJF (SRTF) Example

Same processes. CPU preempts whenever a shorter job arrives.

```
t=0: Only P1 ready → run P1
t=1: P2 arrives (BT=4). P1 remaining=5 > 4 → preempt! Run P2
t=2: P3 arrives (BT=2). P2 remaining=3 > 2 → preempt! Run P3
t=3: P4 arrives (BT=5). P3 remaining=1 < 5 → continue P3
t=4: P3 done. Ready: P1(5), P2(3), P4(5) → run P2 (shortest)
t=7: P2 done. Ready: P1(5), P4(5) → run P1 (earlier arrival)
t=12: P1 done → run P4
t=17: P4 done
```

**Gantt Chart:**
```
|P1|P2|P3|P3|P2|P2|P2|P1|P1|P1|P1|P1|P4|P4|P4|P4|P4|
0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17
```

### Properties

| Property | Detail |
|----------|--------|
| Preemptive? | Optional |
| Optimal? |  Minimum avg WT (non-preemptive) |
| Starvation |  Yes — long jobs may never run |
| Problem | BT not known in advance (CPU burst prediction needed) |

---

## 3.  Round Robin (RR)

### How It Works
Each process gets a fixed **time quantum (q)**. After q, it goes back to the end of the Ready queue. **Preemptive**.

```
Time quantum q = 2

Ready queue: P1(5) → P2(3) → P3(4)

| P1(2) | P2(2) | P3(2) | P1(2) | P2(1) | P3(2) | P1(1) |
0       2       4       6       8       9      11      12
```

### Full Example (q = 2)

| Process | AT | BT |
|---------|----|----|
| P1 | 0 | 5 |
| P2 | 0 | 3 |
| P3 | 0 | 4 |

**Gantt Chart:**
```
|P1|P1|P2|P2|P3|P3|P1|P1|P2|P3|P3|P1|
0  2  4  6  8 10 12 14 15 17 19 20
```

| Process | BT | CT | TAT | WT |
|---------|----|----|----|----|
| P1 | 5 | 20 | 20 | 15 |
| P2 | 3 | 15 | 15 | 12 |
| P3 | 4 | 19 | 19 | 15 |

### Effect of Time Quantum Size

| q too small | q too large |
|-------------|-------------|
| Too many context switches (overhead) | Degenerates into FCFS |
| Good response time | Poor response time |

**Rule of thumb**: q should be larger than 80% of CPU bursts.

### Properties

| Property | Detail |
|----------|--------|
| Preemptive? |  Yes |
| Starvation |  No — everyone gets equal turns |
| Response time |  Good for interactive systems |
| Overhead | More context switches |

---

## 4.  Priority Scheduling

### How It Works
Each process has a **priority**. Highest priority runs first.  
Can be **preemptive** or **non-preemptive**.

```
Convention: Lower number = Higher priority (P0 > P1 > P2...)
            OR Higher number = Higher priority (depends on system)
```

### Example (Non-preemptive, lower = higher priority)

| Process | AT | BT | Priority |
|---------|----|----|----------|
| P1 | 0 | 5 | 3 |
| P2 | 0 | 3 | 1 |
| P3 | 0 | 4 | 2 |

Order: P2 (priority 1) → P3 (priority 2) → P1 (priority 3)

**Gantt Chart:**
```
| P2(3) | P3(4) | P1(5) |
0       3       7      12
```

### Problem: Starvation
Low-priority processes may **never run** if high-priority processes keep arriving.

**Solution: Aging** — gradually increase priority of waiting processes (see [03-Starvation-and-Aging.md](./03-Starvation-and-Aging.md))

---

## 5. Multilevel Queue Scheduling (MLQ)

> Processes are permanently assigned to a **fixed queue** based on their type. Each queue has its own scheduling algorithm and its own priority.

```
Queue 0 (Highest priority) — System processes      → FCFS
Queue 1                    — Interactive processes  → Round Robin
Queue 2                    — Batch processes        → SJF
Queue 3 (Lowest priority)  — Background jobs        → FCFS

CPU always picks from Queue 0 first.
Queue 1 only gets CPU when Queue 0 is empty.
Queue 2 only gets CPU when Queue 0 and Queue 1 are empty.
```

- **Fixed assignment** — a process cannot move between queues
- **No starvation handling** — low-priority queues may starve if high-priority queues are always busy
- Simple to implement — each queue independently scheduled

| Characteristic | Detail |
|---------------|--------|
| Queue assignment | Permanent (fixed by process type) |
| Between-queue scheduling | Priority-based (strict or time-sliced) |
| Starvation | Possible for low-priority queues |

---

## 6. Multilevel Feedback Queue Scheduling (MLFQ)

> Like MLQ, but processes **can move between queues** based on their CPU usage behavior. CPU-bound processes are demoted; I/O-bound processes are promoted.

```
Queue 0: Round Robin (q=8ms)    ← highest priority, short quantum
Queue 1: Round Robin (q=16ms)   ← medium priority
Queue 2: FCFS                   ← lowest priority, unlimited quantum

New process enters Queue 0.

If it uses full 8ms quantum without blocking:
  → Demoted to Queue 1 (suspected CPU-bound)

If it uses full 16ms in Queue 1:
  → Demoted to Queue 2 (confirmed CPU-bound, gets FCFS)

If it blocks for I/O before quantum expires:
  → Stays in same queue or promoted (I/O-bound, responsive)
```

### Key Parameters

| Parameter | Description |
|-----------|-------------|
| Number of queues | Typically 3–8 |
| Scheduling within each queue | Usually Round Robin (top queues), FCFS (bottom) |
| Demotion rule | If process uses full quantum → move to lower queue |
| Promotion rule | If process waits too long → move to higher queue (aging) |
| Quantum per queue | Increases at lower queues (2x rule common) |

### Why MLFQ is Powerful

- Short, interactive processes (I/O-bound) stay in high-priority queues → fast response
- Long CPU-bound jobs sink to low-priority queues → don't starve interactive jobs
- Aging prevents starvation of CPU-bound processes
- No need to know burst time in advance (unlike SJF)

---

## Algorithm Comparison Table

| Algorithm | Preemptive | Starvation | Avg WT | Best For |
|-----------|-----------|-----------|--------|----------|
| FCFS | No | No | High | Simple batch |
| SJF (non-pre) | No | Yes | Minimum | Known burst times |
| SRTF | Yes | Yes | Optimal | Minimize wait time |
| Round Robin | Yes | No | Medium | Interactive systems |
| Priority | Both | Yes (low) | Medium | Real-time |
| MLQ | Both | Yes (low queues) | Varies | Fixed process categories |
| MLFQ | Yes | No (with aging) | Good | General purpose OS |

---

## Interview Questions & Answers

**Q: Which scheduling algorithm gives minimum average waiting time?**
> SJF (Shortest Job First) / SRTF gives provably minimum average waiting time. However, it requires knowing burst time in advance, which is not always possible.

**Q: What is the convoy effect in FCFS?**
> When a long process holds the CPU, all shorter processes behind it wait a long time — like cars behind a slow convoy. This leads to high average waiting time. SJF and RR avoid this.

**Q: How do you calculate turnaround time and waiting time?**
> TAT = Completion Time − Arrival Time. WT = TAT − Burst Time. Draw the Gantt chart first, then fill in completion times for each process.

**Q: What is the trade-off in choosing time quantum for Round Robin?**
> Small q = better response time but more context switch overhead. Large q = less overhead but poor response time (degenerates to FCFS). Optimal q should be slightly larger than a typical CPU burst.

**Q: What is the difference between MLQ and MLFQ?**
> In MLQ, processes are permanently assigned to a fixed queue based on type and cannot move between queues — a batch job stays in the batch queue forever. In MLFQ, processes dynamically move between queues based on behavior: CPU-bound processes get demoted to lower-priority queues, I/O-bound processes stay in higher-priority queues. MLFQ is more adaptive and is used in most real OSes (Windows, macOS, Linux).

**Q: Why is MLFQ considered the best general-purpose scheduler?**
> MLFQ doesn't require knowing burst times (unlike SJF), prevents starvation through aging, gives fast response to interactive I/O-bound processes, and allows long CPU-bound jobs to complete without disrupting interactive work. It approximates the behavior of SJF without needing future knowledge.

---

*← [Topic 2 Index](./README.md) | Next → [Context Switching](./02-Context-Switching.md)*
