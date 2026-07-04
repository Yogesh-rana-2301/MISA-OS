# 📅 CPU Scheduling Algorithms

---

## Why Scheduling?

When multiple processes are **Ready**, the OS must decide which one runs next on the CPU.  
The **CPU Scheduler** makes this decision.

---

## 📐 Key Metrics (Know These Cold)

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

## 1. 📋 FCFS — First Come First Served

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
| Preemptive? | ❌ No |
| Simple? | ✅ Very easy to implement |
| Convoy Effect | ✅ Yes — short jobs wait behind long ones |
| Starvation | ❌ No — everyone eventually runs |

### Convoy Effect
```
Long job P1 (BT=100) arrives at t=0
Short jobs P2, P3, P4 (BT=1 each) arrive at t=1
→ P2, P3, P4 all wait 100ms for P1 to finish!
```

---

## 2. ⚡ SJF — Shortest Job First

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
| Optimal? | ✅ Minimum avg WT (non-preemptive) |
| Starvation | ✅ Yes — long jobs may never run |
| Problem | BT not known in advance (CPU burst prediction needed) |

---

## 3. 🔄 Round Robin (RR)

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
| Preemptive? | ✅ Yes |
| Starvation | ❌ No — everyone gets equal turns |
| Response time | ✅ Good for interactive systems |
| Overhead | More context switches |

---

## 4. 🏆 Priority Scheduling

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

## 📊 Algorithm Comparison Table

| Algorithm | Preemptive | Starvation | Avg WT | Overhead | Best For |
|-----------|-----------|-----------|--------|----------|----------|
| FCFS | ❌ | ❌ | High | Low | Simple batch jobs |
| SJF (non-pre) | ❌ | ✅ | Minimum | Low | Known burst times |
| SRTF (pre-SJF) | ✅ | ✅ | Optimal | Medium | Minimizing wait time |
| Round Robin | ✅ | ❌ | Medium | High | Interactive systems |
| Priority | Both | ✅ (low pri) | Medium | Medium | Real-time, prioritized |

---

## 🎯 Interview Questions & Answers

**Q: Which scheduling algorithm gives minimum average waiting time?**
> SJF (Shortest Job First) / SRTF gives provably minimum average waiting time. However, it requires knowing burst time in advance, which is not always possible.

**Q: What is the convoy effect in FCFS?**
> When a long process holds the CPU, all shorter processes behind it wait a long time — like cars behind a slow convoy. This leads to high average waiting time. SJF and RR avoid this.

**Q: How do you calculate turnaround time and waiting time?**
> TAT = Completion Time − Arrival Time. WT = TAT − Burst Time. Draw the Gantt chart first, then fill in completion times for each process.

**Q: What is the trade-off in choosing time quantum for Round Robin?**
> Small q = better response time but more context switch overhead. Large q = less overhead but poor response time (degenerates to FCFS). Optimal q should be slightly larger than a typical CPU burst.

**Q: What is the difference between preemptive and non-preemptive scheduling?**
> Preemptive: The OS can interrupt a running process (via timer or higher-priority arrival). Non-preemptive: A process runs until it voluntarily gives up the CPU (completes or blocks on I/O).

---

*← [Topic 2 Index](./README.md) | Next → [Context Switching](./02-Context-Switching.md)*
