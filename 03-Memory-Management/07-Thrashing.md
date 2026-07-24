#  Thrashing & Working Set Model

---

## What is Thrashing?

**Thrashing** is a condition where the CPU spends **more time handling page faults than executing actual process instructions**.

> The system is so busy swapping pages in and out of disk that no real work gets done.

```
Normal:
CPU utilization:  ████████████████████░░░░  (~85% useful work)
Page fault I/O:   ░░░░░░░░░░░░░░░░░░░░████

Thrashing:
CPU utilization:  ██░░░░░░░░░░░░░░░░░░░░░░  (~10% useful work)
Page fault I/O:   ░░████████████████████████  (constantly swapping!)
```

---

## How Thrashing Happens — Step by Step

```
1. Many processes in memory → each gets very few frames
2. Each process needs more pages than it has frames
3. Frequent page faults → processes spend time waiting for disk
4. CPU appears free → OS thinks: "load MORE processes!" (wrong!)
5. More processes → even fewer frames per process
6. Even more page faults → CPU utilization collapses
7. System is in THRASHING
```

### The Thrashing Curve

```
CPU Utilization
    │
    │          optimal
    │       ╱   ╲
    │     ╱       ╲
    │   ╱           ╲ ← THRASHING begins here
    │ ╱               ╲
    └──────────────────────────────→ Degree of Multiprogramming
                         ↑
               Too many processes for available RAM
```

---

## Cause & Effect

| Cause | Effect |
|-------|--------|
| Too many processes in RAM | Too few frames per process |
| Too few frames per process | Frequent page faults |
| Frequent page faults | Process always waiting for disk |
| Process always waiting | CPU appears idle |
| OS adds more processes | Thrashing intensifies |

---

## Working Set Model (Solution to Thrashing)

### The Idea

The **working set** of a process is the set of pages it **actively uses** during a given time window Δ.

```
Working Set Window: Δ = 10 (last 10 references)

Reference string: 1,2,3,2,1,4,2,1,3,4 | 5,2,3,5,2

Working set at t1 (references 1–10): {1, 2, 3, 4}  → needs 4 frames
Working set at t2 (references 6–15): {2, 3, 4, 5}  → needs 4 frames
```

### The Rule

```
Total demand D = Σ working_set_size(process_i)

If D > total frames available:
    → Thrashing possible → SUSPEND a process (free its frames)

If D ≤ total frames:
    → Safe → can add more processes
```

### Why It Works

- Each process only keeps pages it's **currently using** in RAM
- No process starves another of frames it needs
- If total demand exceeds RAM, reduce multiprogramming (suspend a process)

---

## Preventing Thrashing

| Strategy | How |
|----------|-----|
| **Working Set Model** | Track active page set; don't let D > total frames |
| **Page Fault Frequency (PFF)** | Monitor fault rate: too high → give more frames; too low → take frames back |
| **Limit multiprogramming** | Don't load more processes than RAM can support |
| **Increase RAM** | More frames → each process gets more → fewer faults |
| **Swapping** | Temporarily swap out entire processes to free frames |

---

## Thrashing vs Starvation vs Deadlock (Quick Comparison)

| | Thrashing | Starvation | Deadlock |
|-|-----------|------------|---------|
| **Who suffers** | All processes in system | One/few processes | Specific group |
| **Root cause** | Too little RAM per process | Unfair scheduling | Circular resource wait |
| **Makes progress?** | Very little | Others do | None |
| **Solution** | Reduce multiprogramming | Aging | Prevention/avoidance |

---

##  Interview Questions & Answers

**Q: What is thrashing?**
> Thrashing occurs when the OS spends more time swapping pages in/out of disk than executing process instructions. CPU utilization collapses. It's caused by too many processes competing for too few frames — each process gets fewer frames than its working set requires, causing constant page faults.

**Q: What causes thrashing?**
> When the degree of multiprogramming is too high — more processes are loaded than the available RAM can support. Each process gets fewer frames than it needs, leading to constant page faults, disk I/O, and near-zero CPU utilization.

**Q: What is the working set model?**
> The working set of a process is the set of pages it actively uses in the last Δ references (the working set window). The OS tracks working sets and ensures total demand (sum of all working set sizes) doesn't exceed available frames. If it does, a process is suspended to prevent thrashing.

**Q: How does the OS detect and recover from thrashing?**
> The OS monitors CPU utilization. If CPU utilization drops significantly despite processes being ready, thrashing may be occurring. Recovery: suspend one or more processes (swap them out entirely), freeing their frames for remaining processes, then gradually reload them.

---

*← [Page Replacement](./06-Page-Replacement.md) | Next → [Cache Memory & TLB](./08-Cache-and-TLB.md)*
